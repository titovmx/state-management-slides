<style>
.two-column-container {
  display: flex;
  gap: 1rem;
}

.two-column-container .left {
  flex: 0 1 30%;
}
</style>

<template>
  <div class="slidev-layout two-column-code">
    <div>
      <slot name="default" />
    </div>
    <div class="two-column-container">
      <div class="left">
        <slot name="left" />
      </div>
      <div class="right">
        <slot name="right" />
      </div>
    </div>
  </div>
</template>
