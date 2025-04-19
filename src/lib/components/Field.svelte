<script>
    export let label = '';
    export let id = '';
    export let type = 'text';
    export let value = '';
    export let error = '';
  </script>
  
  <div class="field">
    <label for={id}>{label}</label>
    <input
      id={id}
      type={type}
      class:error={!!error}
      bind:value={value}
    />
    {#if error}
      <span class="field-error">{error}</span>
    {/if}
  </div>
  
  <style>
    .field {
      margin-bottom: 1rem;
      display: flex;
      flex-direction: column;
    }
  
    label {
      font-weight: bold;
    }
  
    input.error {
      border-color: red;
    }
  
    .field-error {
      color: red;
      font-size: 0.875rem;
      margin-top: 2px;
    }
  </style>
  