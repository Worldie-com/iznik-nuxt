<template>
  <b-modal
    id="forgetfailmodal"
    v-model="showModal"
    title="Sorry, that didn't work"
    no-stacking
  >
    <template slot="default">
      <b-row>
        <b-col>
          <p>
            <!-- eslint-disable-next-line -->
            Please contact <ExternalLink href="mailto:support@ilovefreegle.org">support@ilovefreegle.org</ExternalLink> if you need further help.
          </p>
        </b-col>
      </b-row>
    </template>
    <template slot="modal-footer" slot-scope="{ ok, cancel }">
      <b-button variant="white" @click="cancel">
        Close
      </b-button>
    </template>
  </b-modal>
</template>
<script>
import modal from '@/mixins/modal'
const ExternalLink = () => import('~/components/ExternalLink')

export default {
  components: { ExternalLink },
  mixins: [modal],
  methods: {
    confirm() {
      this.$emit('confirm')
      this.hide()
    }
  }
}
</script>
