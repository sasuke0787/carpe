<script lang="ts">
  import { onMount } from 'svelte'
  import { _ } from 'svelte-i18n'
  import { invoke } from '@tauri-apps/api/tauri'
  import { formatAccount } from '../../modules/accounts'
  import { notify_success } from '../../modules/carpeNotify'
  import { raise_error } from '../../modules/carpeError'
  import { refreshAccounts } from '../../modules/accountActions'
  import type { CarpeProfile } from '../../modules/accounts'
  
  export let account: CarpeProfile
  export let watchOnly: boolean
  
  let receiver: string = ''
  let errorMessage = ''
  let waitingTxs = false
  const re = /[a-fA-F0-9]{32}/i

  // Vouch limit variables
  let vouchLimit = 0
  let isLoadingVouchData = false

  let vouchedAccounts: string[] = []
  let isLoadingVouchedAccounts = false
  
  // Validation for the receiver address
  $: isReceiverValid = account && receiver && re.test(receiver) && receiver != account.account

  onMount(async () => {
    if (account) {
      fetchVouchLimitData()
      fetchVouchedAccounts()
    }
  })

  // Add this function to fetch the list of vouched accounts
  async function fetchVouchedAccounts() {
    if (!account) return
    
    isLoadingVouchedAccounts = true
    try {
      vouchedAccounts = await invoke('get_given_vouches', {
        account: account.account
      })
    } catch (error) {
      console.error('Error fetching vouched accounts:', error)
    } finally {
      isLoadingVouchedAccounts = false
    }
  }

  // Fetch vouch limit data
  async function fetchVouchLimitData() {
    if (!account) return
    
    isLoadingVouchData = true
    try {
      // Get the vouch limit
      vouchLimit = await invoke('get_vouch_limit', {
        account: account.account
      })
    } catch (error) {
      console.error('Error fetching vouch data:', error)
    } finally {
      isLoadingVouchData = false
    }
  }
  
  // Refresh vouch data
  async function refreshVouchData() {
    await fetchVouchLimitData()
    await fetchVouchedAccounts()
  }
  
  // Handle vouch transaction
  async function submitVouch() {
    if (!account || !receiver || !isReceiverValid) {
      errorMessage = $_('txs.vouch.invalid_address')
      return
    }
    
    waitingTxs = true
    errorMessage = ''
    
    try {
      console.log(`Submitting vouch transaction from ${account.account} to ${receiver}`)
      
      await invoke("vouch_transaction", {
        sender: account.account,
        receiver: receiver,
        legacy: false
      })
      
      notify_success($_('txs.vouch.success'))
      receiver = ''
      
      // Refresh vouch data after successful transaction
      await refreshVouchData()
      refreshAccounts()
    } catch (e) {
      console.error("Vouch failed:", e)
      errorMessage = $_('txs.vouch.invalid_address')
      raise_error(e, true, "vouch_transaction")
    } finally {
      waitingTxs = false
    }
  }
</script>

<!-- Sender and Vouch Limit Info -->
<div class="uk-grid-small uk-margin-bottom" uk-grid>
  <div class="uk-width-3-4@s">
    <label class="uk-form-label" for="vouch-sender">{$_('txs.vouch.sender')}</label>
    <div class="uk-form-controls uk-form-controls-text">
      <!-- Display shortened address with copy functionality if available -->
      <div class="sender-address">
        {#if account?.account}
          <span id="vouch-sender">{formatAccount(account.account)}</span>
        {/if}
      </div>
    </div>
  </div>
  <div class="uk-width-1-4@s">
    <span class="uk-form-label" id="vouch-limit-label">
      {$_('txs.vouch.vouch_limit')}
      {#if isLoadingVouchData}
        <span uk-spinner="ratio: 0.5"></span>
      {/if}
    </span>
    <div class="uk-flex uk-flex-middle" aria-labelledby="vouch-limit-label">
      <span>{vouchLimit}</span>
      <button 
        class="uk-button uk-button-small uk-button-text uk-margin-small-left"
        uk-tooltip={$_('txs.vouch.refresh_tooltip')}
        on:click={refreshVouchData}
        disabled={isLoadingVouchData}>
        <span uk-icon="icon: refresh; ratio: 0.7"></span>
      </button>
    </div>
  </div>
</div>

<!-- Receiver input -->
<div class="uk-margin">
  <label class="uk-form-label" for="vouch-receiver">{$_('txs.vouch.receiver')}</label>
  <div class="uk-form-controls">
    <input
      id="vouch-receiver"
      class="uk-input {!isReceiverValid && receiver ? 'uk-form-danger' : ''}"
      type="text"
      bind:value={receiver}
      placeholder={$_('txs.vouch.receiver_placeholder')}
      disabled={watchOnly || waitingTxs || vouchLimit <= 0}
    />
    {#if !isReceiverValid && receiver}
      <p class="uk-text-danger uk-text-small">{$_('txs.vouch.invalid_address')}</p>
    {/if}
  </div>
</div>

<div class="uk-margin">
  <button
    class="uk-button uk-button-primary uk-width-1-1"
    on:click={submitVouch}
    disabled={!isReceiverValid || !receiver || watchOnly || waitingTxs || vouchLimit <= 0}
  >
    {#if waitingTxs}
      <span uk-spinner="ratio: 0.6"></span> {$_('txs.vouch.await')}
    {:else}
      {$_('txs.vouch.btn_vouch')}
    {/if}
  </button>
  
  {#if vouchLimit <= 0}
    <p class="uk-text-warning uk-text-small">
      {$_('txs.vouch.limit_reached')}
    </p>
  {/if}
</div>

{#if errorMessage}
  <div class="uk-alert uk-alert-danger">
    <p>{errorMessage}</p>
  </div>
{/if}

<div class="uk-margin-top">
  <button class="uk-button uk-button-default uk-width-1-1 uk-flex uk-flex-between uk-flex-middle" type="button" uk-toggle="target: #vouched-accounts">
    <span>{$_('txs.vouch.view_current_vouches')} ({vouchedAccounts.length})</span>
    <span uk-icon="icon: chevron-down"></span>
  </button>
  
  <div id="vouched-accounts" hidden>
    <div class="uk-card uk-card-default uk-card-body uk-padding-small uk-margin-small-top">
      {#if isLoadingVouchedAccounts}
        <div class="uk-flex uk-flex-center uk-margin">
          <div uk-spinner="ratio: 0.8"></div>
          <span class="uk-margin-small-left">{$_('txs.vouch.loading_vouched_accounts')}</span>
        </div>
      {:else if vouchedAccounts.length === 0}
        <p class="uk-text-center uk-text-muted">{$_('txs.vouch.no_vouched_accounts')}</p>
      {:else}
        <div class="uk-flex uk-flex-between uk-margin-small-bottom">
          <span>{$_('txs.vouch.accounts_vouched_for')}</span>
          <button 
            class="uk-button uk-button-small uk-button-text"
            uk-tooltip={$_('txs.vouch.refresh_vouched_accounts')} 
            on:click={fetchVouchedAccounts}
            disabled={isLoadingVouchedAccounts}>
            <span uk-icon="icon: refresh; ratio: 0.7"></span>
          </button>
        </div>
        
        <div class="uk-height-small uk-overflow-auto">
          <ul class="uk-list uk-list-divider uk-margin-remove">
            {#each vouchedAccounts as address}
              <li class="uk-padding-small uk-padding-remove-horizontal">
                {address}
              </li>
            {/each}
          </ul>
        </div>
      {/if}
    </div>
  </div>
</div>

<div class="uk-margin-top uk-text-meta">
  <h4>{$_('txs.vouch.important_info')}</h4>
  <ul class="uk-list uk-list-bullet">
    <li>{$_('txs.vouch.info_relationship')}</li>
    <li>{$_('txs.vouch.info_selective')}</li>
    <li>{$_('txs.vouch.info_limit')}</li>
  </ul>
</div>
