<template>
  <input
    v-model="stringValue"
    :name="name"
    class="form-control"
    :class="{ 'is-invalid': isInvalid }"
    required="required"
    size="16"
    type="date"
    :disabled="!enabled"
    :readonly="usedFallback"
    :max="max"
  />
</template>

<script lang="ts">
import { Vue, Component, Watch, Prop } from 'vue-property-decorator';
import T from '../lang';
import * as time from '../time';
import '../../../third_party/js/bootstrap-datepicker.js';

@Component
export default class DatePicker extends Vue {
  T = T;

  @Prop({ default: '' }) name!: string;
  @Prop() value!: Date;
  @Prop({ default: true }) enabled!: boolean;
  @Prop({ default: T.datePickerFormat }) format!: string;
  @Prop({ default: false }) isInvalid!: boolean;
  @Prop({ default: '' }) max!: string;

  private usedFallback: boolean = false;
  private stringValue: string = time.formatDateLocal(this.value);

  mounted() {
    if ((this.$el as HTMLInputElement).type === 'text') {
      // Even though we declared the input as having date type,
      // browsers that don't support it will silently change the type to text.
      // In that case, use the bootstrap datepicker.
      this.mountedFallback();
    }
  }

  private mountedFallback() {
    this.usedFallback = true;
    $(this.$el)
      .datepicker({
        weekStart: 1,
        format: this.format,
      })
      .on('changeDate', (ev) => {
        this.$emit('input', ev.date);
      })
      .datepicker('setValue', this.value);
  }

  @Watch('stringValue')
  onStringValueChanged(newStringValue: string) {
    if (this.usedFallback) {
      // If the fallback was used, we don't need to update anything.
      return;
    }
    this.$emit('input', time.parseDateLocal(newStringValue));
  }

  @Watch('value')
  onPropertyChanged(newValue: Date) {
    this.stringValue = time.formatDateLocal(newValue);
    if (this.usedFallback) {
      $(this.$el).datepicker('setValue', newValue);
    }
  }
}
</script>

<style>
@import '../../../third_party/css/bootstrap-datepicker.css';
</style>
