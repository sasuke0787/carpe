<script lang="ts">
  import { _ } from 'svelte-i18n';
  import { createEventDispatcher } from 'svelte';
  
  // Props
  export let showVouchInstructions = false;
  
  // Event dispatcher for close button
  const dispatch = createEventDispatcher();
  
  function closeCard() {
    dispatch('close');
  }
</script>

{#if showVouchInstructions}
  <div class="uk-card uk-card-primary uk-card-small uk-margin">
    <div class="uk-card-header">
      <div class="uk-flex uk-flex-middle">
        <h3 class="uk-card-title uk-flex-1">{$_('wallet.vouch_instructions.title')}</h3>
        <button 
          class="uk-close-large" 
          uk-close 
          on:click={closeCard}
        ></button>
      </div>
    </div>
    <div class="uk-card-body">
      <p>{$_('wallet.vouch_instructions.description')}</p>
      
      <h4 class="uk-text-bold uk-margin-small-bottom">{$_('wallet.vouch_instructions.what_you_need')}</h4>
      <ul class="uk-list uk-list-bullet">
        <li>{$_('wallet.vouch_instructions.need_item_1')}</li>
        <li>{$_('wallet.vouch_instructions.need_item_2')}</li>
      </ul>
      
      <h4 class="uk-text-bold uk-margin-small-bottom">{$_('wallet.vouch_instructions.how_to_get')}</h4>
      <ol class="uk-list uk-list-numbered">
        <li>{$_('wallet.vouch_instructions.step_1')}</li>
        <li>{$_('wallet.vouch_instructions.step_2')}</li>
        <li>{$_('wallet.vouch_instructions.step_3')}</li>
      </ol>
      
      <p class="uk-margin-top">
        {$_('wallet.vouch_instructions.learn_more')}
        <a href="https://docs.openlibra.io/guides/founder/migrate-to-v8" target="_blank" rel="noopener noreferrer">
          {$_('wallet.vouch_instructions.documentation')}
        </a>
      </p>
    </div>
  </div>
{/if}

<style>
  .uk-card-primary {
    background: linear-gradient(to right, #4a148c, #7b1fa2);
    color: white;
    border-radius: 8px;
    box-shadow: 0 4px 15px rgba(123, 31, 162, 0.3);
    transition: all 0.3s ease;
  }
  
  .uk-card-primary:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(123, 31, 162, 0.4);
  }
  
  .uk-card-header {
    border-bottom: 1px solid rgba(255, 255, 255, 0.2);
  }
  
  .uk-card-title {
    font-weight: 600;
    color: white;
    font-size: 1.4rem;
  }
  
  h4 {
    color: white;
    opacity: 0.9;
    font-size: 1.1rem;
    margin-top: 16px;
  }
  
  .uk-list li {
    margin-bottom: 10px;
    line-height: 1.5;
  }
  
  a {
    color: #ffeb3b;
    text-decoration: underline;
    transition: color 0.2s ease;
  }
  
  a:hover {
    color: #fff176;
    text-decoration: none;
  }
</style>
