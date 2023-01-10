<script lang="ts">
  export let label: string
</script>

<label class="row">
  <div class="col-3">
    { label }
  </div>
  <div class="col-9">
    <slot></slot>
  </div>
</label>

<style>
  label {
    margin-bottom: 10px;
  }
</style>