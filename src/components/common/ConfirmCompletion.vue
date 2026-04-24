<template>
  <section
    id="completing-party-stmnt-section"
  >
    <header>
      <slot name="header" />
      <p class="mt-4">
        The following information must be completed and confirmed before submitting this filing.
      </p>
    </header>

    <v-card
      flat
      class="mt-6 py-8 px-6"
      :class="{ 'invalid-section': invalidSection }"
    >
      <v-form
        lazy-validation
        @submit.prevent
      >
        <v-row no-gutters>
          <v-col
            cols="12"
            sm="3"
          >
            <label
              class="title-label"
              :class="{'error-text': invalidSection}"
            >
              Confirm Completion
            </label>
          </v-col>
          <v-col
            class="px-5 py-5 gray-background"
            cols="12"
            sm="9"
          >
            <v-row class="no-gutters">
              <v-checkbox
                :value="getConfirmCompletionState.confirmed"
                hide-details
                class="mt-0 pt-0"
                @change="emitConfirmed($event)"
              >
                <template #label>
                  <p
                    class="stmt-text"
                    :class="{'error-text': invalidSection}"
                  >
                    <slot name="default" />
                  </p>
                </template>
              </v-checkbox>
            </v-row>
            <v-row class="no-gutters pl-7">
              <p class="stmt-text">
                <slot name="under-checkbox" />
              </p>
            </v-row>
          </v-col>
        </v-row>
      </v-form>
    </v-card>
  </section>
</template>

<script lang="ts">
import { Component, Emit, Prop, Vue } from 'vue-property-decorator'
import { Getter } from 'pinia-class'
import { useStore } from '@/store/store'
import { ConfirmCompletionIF } from '@/interfaces'

@Component({})
export default class ConfirmCompletion extends Vue {
  @Getter(useStore) getConfirmCompletionState!: ConfirmCompletionIF

  @Prop({ required: true }) readonly heading!: string
  @Prop({ default: false }) readonly invalidSection!: boolean

  @Emit('emitConfirmed')
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  emitConfirmed (confirmed: boolean): void {}
}
</script>

<style lang="scss" scoped>
@import "@/assets/styles/theme.scss";

.stmt-text {
  display: inline;
  font-size: 0.875rem;
  color: $gray7;
  font-weight: normal;
}

.title-label {
  font-weight: bold;
  color: $gray9;
}

// align checkbox with top of its label
:deep(.v-input--checkbox .v-input__slot) {
  align-items: flex-start;
}
</style>
