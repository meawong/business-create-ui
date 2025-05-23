<template>
  <div id="amalgamation-share-structure">
    <section class="mt-10">
      <header>
        <h2>1. Create Your Authorized Share Structure</h2>
      </header>

      <p>
        Add at least one class of shares. A share class consists of the name of the class, the
        maximum number of shares in the class (including any associated Series), a par value for
        the class, and the currency the shares are valued in.
      </p>
    </section>

    <!-- Help Section -->
    <span
      class="help-btn"
      @click="helpToggle = !helpToggle"
    >
      <v-icon
        color="primary"
        style="padding-right: 5px"
      >mdi-help-circle-outline</v-icon>
      <span v-if="!helpToggle">Help with Share Structure</span>
      <span v-else>Hide Help</span>
    </span>

    <v-expand-transition>
      <section
        v-show="helpToggle"
        class="share-structure-help"
      >
        <header id="share-structure-help-header">
          <h2>Share Structure Help</h2>
        </header>

        <p>
          An amalgamated business must issue shares. These shares represent ownership interest in
          the company and give the shareholder a say in how the company is being run. For most
          small companies starting out, a simple share structure with just one class of shares (and
          no series) is typical.
        </p>
        <p>
          The sample below is for a typical share structure with just one class of shares. If there
          is more than one class of shares, each class must be assigned an identifying name such as
          Class A, Class B, etc.
        </p>

        <v-card flat>
          <header class="share-summary-header">
            <v-icon color="appDkBlue">
              mdi-sitemap
            </v-icon>
            <label class="share-summary-header-title pl-2">Share Structure</label>
          </header>

          <ListShareClass
            :shareClasses="sharesHelpSample"
            :isSummary="true"
          />
        </v-card>

        <p><small>Sample Share Structure</small></p>
        <h3>Important Information About the Company's Share Structure</h3>
        <p>
          The staff at the Corporate Registry cannot provide advice on how to set up your company's
          share structure. If you do not understand what an authorized share structure is or what
          its purpose is or believe you need a more complex share structure, you should seek
          professional advice or purchase an amalgamation guide for detailed information and
          infrastructure on establishing an authorized share structure.
        </p>
        <p>
          Refer to this <a :href="helpLink">link</a> to obtain more information on amalgamating a company.
        </p>
        <u
          class="help-btn"
          @click="helpToggle = !helpToggle"
        ><small>Hide Help</small></u>
      </section>
    </v-expand-transition>

    <div class="mt-4 font-weight-bold">
      Your application must include the following:
    </div>
    <ul class="mt-2">
      <li>
        <v-icon
          v-if="shareClasses.length > 0"
          color="green darken-2"
          class="cp-valid"
        >
          mdi-check
        </v-icon>
        <v-icon
          v-else-if="getShowErrors"
          color="error"
          class="cp-invalid"
        >
          mdi-close
        </v-icon>
        <v-icon v-else>
          mdi-circle-small
        </v-icon>
        <span class="ml-2">At least one Class of Shares</span>
      </li>
    </ul>

    <div class="py-8">
      <v-btn
        id="btn-start-add-cp"
        outlined
        color="primary"
        :disabled="showShareStructureForm"
        @click="initNewShareClass()"
      >
        <v-icon>mdi-plus</v-icon>
        <span>Add Share Class</span>
      </v-btn>
    </div>

    <v-card
      v-if="showShareStructureForm"
      flat
      class="my-4"
    >
      <ShareStructure
        :initialValue="currentShareStructure"
        :activeIndex="activeIndex"
        :shareId="shareId"
        :parentIndex="parentIndex"
        :shareClasses="shareClasses"
        @addEditClass="addEditShareClass($event)"
        @addEditSeries="addEditShareSeries($event)"
        @removeClass="removeShareClass($event)"
        @removeSeries="removeSeries"
        @resetEvent="resetData()"
      />
    </v-card>

    <v-card
      v-if="shareClasses.length > 0"
      flat
    >
      <ListShareClass
        :shareClasses="shareClasses"
        :componentDisabled="showShareStructureForm"
        @editClass="initShareClassForEdit($event)"
        @removeClass="removeShareClass($event)"
        @addSeries="initNewShareSeries($event)"
        @editSeries="editSeries"
        @removeSeries="removeSeries"
      />
    </v-card>
  </div>
</template>

<script lang="ts">
import { Component, Mixins, Watch } from 'vue-property-decorator'
import { Action, Getter } from 'pinia-class'
import { useStore } from '@/store/store'
import { v4 as uuidv4 } from 'uuid'
import { NewShareClass, NewShareSeries, ShareClassIF, ShareStructureIF }
  from '@/interfaces'
import { CommonMixin } from '@/mixins'
import { RouteNames } from '@/enums'
import ListShareClass from '@/components/common/ListShareClass.vue'
import ShareStructure from '@/components/common/ShareStructure.vue'

@Component({
  components: {
    ListShareClass,
    ShareStructure
  }
})
export default class AmalgamationShareStructure extends Mixins(CommonMixin) {
  @Getter(useStore) getCreateShareStructureStep!: ShareStructureIF
  @Getter(useStore) getShowErrors!: boolean

  @Action(useStore) setCreateShareStructureStepValidity!: (x: boolean) => void
  @Action(useStore) setShareClasses!: (x: ShareClassIF[]) => void

  readonly sharesHelpSample: ShareClassIF[] = [{
    id: '1',
    priority: 0,
    type: 'Class',
    name: 'Common Shares',
    hasMaximumShares: true,
    maxNumberOfShares: 10000,
    hasParValue: false,
    parValue: null,
    currency: null,
    hasRightsOrRestrictions: false,
    series: []
  }]

  readonly helpLink = 'https://www2.gov.bc.ca/gov/content/employment-business/business/' +
    'managing-a-business/permits-licences/businesses-incorporated-companies/incorporated-companies#amalgamate'

  showShareStructureForm = false
  currentShareStructure = null as ShareClassIF
  activeIndex = -1
  parentIndex = -1
  shareId = ''
  helpToggle = false

  get shareClasses (): ShareClassIF[] {
    return this.getCreateShareStructureStep.shareClasses
  }

  /** Called when component is mounted. */
  mounted (): void {
    this.setCreateShareStructureStepValidity(this.shareClasses.length > 0)
  }

  //
  // Event Handlers
  //

  initNewShareClass (): void {
    this.currentShareStructure = { ...NewShareClass }
    this.currentShareStructure.priority =
    this.shareClasses.length === 0 ? 1 : this.shareClasses[this.shareClasses.length - 1].priority + 1
    this.activeIndex = -1
    this.parentIndex = -1
    this.shareId = uuidv4()
    this.showShareStructureForm = true
  }

  initShareClassForEdit (index: number): void {
    this.currentShareStructure = { ...this.shareClasses[index] }
    this.activeIndex = index
    this.parentIndex = -1
    this.showShareStructureForm = true
  }

  initNewShareSeries (shareClassIndex: number): void {
    this.activeIndex = -1
    this.parentIndex = shareClassIndex

    const newList: ShareClassIF[] = [...this.shareClasses]
    const parentShareClass = newList[shareClassIndex]
    const shareSeries = parentShareClass.series
    this.currentShareStructure = { ...NewShareSeries }
    this.currentShareStructure.hasParValue = parentShareClass.hasParValue
    this.currentShareStructure.parValue = parentShareClass.parValue
    this.currentShareStructure.currency = parentShareClass.currency
    this.currentShareStructure.priority =
    shareSeries.length === 0 ? 1 : shareSeries[shareSeries.length - 1].priority + 1
    this.shareId = uuidv4()
    this.showShareStructureForm = true
  }

  addEditShareClass (shareStructure: ShareClassIF): void {
    const newList: ShareClassIF[] = [...this.shareClasses]
    // New Share Structue.
    if (this.activeIndex === -1) {
      newList.push(shareStructure)
    } else {
      // Edit Share Structure.
      newList.splice(this.activeIndex, 1, shareStructure)
    }
    this.setShareClasses(newList)
    this.setCreateShareStructureStepValidity(this.shareClasses.length > 0)
    this.resetData()
  }

  editSeries (index: number, seriesIndex: number): void {
    this.activeIndex = seriesIndex
    this.parentIndex = index
    const newList: ShareClassIF[] = [...this.shareClasses]
    this.currentShareStructure = { ...newList[this.parentIndex].series[this.activeIndex] }
    this.showShareStructureForm = true
  }

  removeSeries (index: number, seriesIndex: number): void {
    this.activeIndex = seriesIndex
    this.parentIndex = index
    const newList: ShareClassIF[] = [...this.shareClasses]
    const parentShareClass = newList[this.parentIndex]
    const series = [...parentShareClass.series]
    series.splice(this.activeIndex, 1)
    parentShareClass.series = series
    this.setShareClasses(newList)
    this.resetData()
  }

  addEditShareSeries (shareSeries: ShareClassIF): void {
    const newList: ShareClassIF[] = [...this.shareClasses]
    const parentShareClass = newList[this.parentIndex]
    const series = [...parentShareClass.series]
    // New Share Structue.
    if (this.activeIndex === -1) {
      series.push(shareSeries)
    } else {
      // Edit Share Structure.
      series.splice(this.activeIndex, 1, shareSeries)
    }
    parentShareClass.series = series
    this.setShareClasses(newList)
    this.resetData()
  }

  removeShareClass (index: number): void {
    const newList: ShareClassIF[] = [...this.shareClasses]
    newList.splice(index, 1)
    this.setShareClasses(newList)
    this.setCreateShareStructureStepValidity(this.shareClasses.length > 0)
    this.resetData()
  }

  resetData (): void {
    this.currentShareStructure = null
    this.activeIndex = -1
    this.showShareStructureForm = false
    this.parentIndex = -1
    this.shareId = ''
  }

  @Watch('$route')
  private async scrollToInvalidComponent (): Promise<void> {
    if (this.getShowErrors && this.$route.name === RouteNames.AMALG_REG_SHARE_STRUCTURE) {
      // scroll to invalid components
      await this.$nextTick()
      await this.validateAndScroll(
        {
          shareStructure: this.getCreateShareStructureStep.valid
        },
        ['amalgamation-share-structure']
      )
    }
  }
}
</script>

<style lang="scss" scoped>
@import '@/assets/styles/theme.scss';

[class^="col"] {
  padding-top: 0;
  padding-bottom: 0;
}

ul {
  padding-top: 0.5rem;
  list-style: none;
  margin-left: 0;
  padding-left: 1rem;
}

li {
  padding-top: 0.25rem;
}

p {
  padding-top: 0.5rem;
}

.help-btn {
  cursor: pointer;
  color: $app-blue;
  vertical-align: middle;
}

.share-structure-help {
  margin: 2rem 0;
  border-top: 1px dashed $gray6;
  border-bottom: 1px dashed $gray6;
  padding: 1rem 0;

  #share-structure-help-header {
    display: flex;
    justify-content: center;
  }

  h2, h4 {
    padding: 1rem 0;
  }

  u {
    display: flex;
    direction: rtl;
  }
}

.share-summary-header {
  display: flex;
  background-color: $BCgovBlue5O;
  padding: 1.25rem;

  .share-summary-header-title {
    font-weight: bold;
    color: $gray9;
  }
}
</style>
