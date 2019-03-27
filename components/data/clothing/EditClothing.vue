<template>
  <edit-data :show="show" name="Clothing" :item="item || null">
    <v-form v-if="item" ref="form" v-model="form">
      <v-layout row wrap>
        <v-flex xs12 px-1>
          <v-select
            v-model="formData.cultures"
            :rules="[rules.required]"
            :items="['Saborian', 'Kioki', 'Marauder', 'Other']"
            label="Culture(s)"
            color="accent"
            multiple
            box
            persistent-hint
          />
        </v-flex>
        <v-flex xs12 px-1>
          <v-select
            v-model="formData.tiers"
            :rules="[rules.required]"
            :items="tiers"
            label="Found in tier(s)"
            color="accent"
            multiple
            chips
            box
            persistent-hint
          />
        </v-flex>
        <v-flex xs6>
          <div class="text-xs-center title">
            Flavor Text Image
          </div>
          <upload-button
            ref="flavor"
            :disabled="!!item.flavor"
            :no-title-update="!!item.flavor"
            title="Uploaded"
            :label-class="`primary--text ${item.flavor ? '' : 'accent'}`"
            width="100%"
            block
            class="px-1"
            @file-update="formData['flavor'] = $event"
          >
            <template v-slot:icon-left>
              <v-icon color="primary">
                attach_file
              </v-icon>
            </template>
          </upload-button>
        </v-flex>
        <v-flex xs6>
          <div class="text-xs-center title">
            Base Image
          </div>
          <upload-button
            ref="base"
            :disabled="!!item.base"
            :no-title-update="!!item.base"
            title="Uploaded"
            label-class="primary--text accent"
            width="100%"
            block
            class="px-1"
            @file-update="formData['base'] = $event"
          >
            <template v-slot:icon-left>
              <v-icon color="primary">
                attach_file
              </v-icon>
            </template>
          </upload-button>
        </v-flex>
      </v-layout>
    </v-form>
    <template v-slot:actions>
      <v-btn color="error" class="secondary--text" @click="$emit('close')">
        Cancel
      </v-btn>
      <v-btn :disabled="!form" :loading="loading" color="success" class="secondary--text" @click="edit">
        Edit
      </v-btn>
    </template>
  </edit-data>
</template>
<script>
import EditData from '@/components/data/EditData'
import UploadButton from '@/components/Upload'

import { rules } from '@/components/mixins/rules'
export default {
  name: 'EditClothing',
  components: {
    EditData,
    UploadButton
  },
  mixins: [rules],
  props: {
    show: Boolean,
    item: {
      type: Object,
      default: () => {}
    }
  },
  data() {
    return {
      form: false,
      loading: false,
      formData: {
        cultures: [],
        tiers: [],
        flavor: null,
        base: null
      },
      tiers: [
        { text: 'Tier 1', value: 1 },
        { text: 'Tier 2', value: 2 },
        { text: 'Tier 3', value: 3 },
        { text: 'Tier 4', value: 4 }
      ]
    }
  },
  watch: {
    item: function(newVal, oldVal) {
      if (newVal === null) {
        this.formData = {
          cultures: [],
          tiers: [],
          flavor: null,
          base: null
        }
      } else {
        this.formData.cultures = newVal.cultures
        this.formData.tiers = newVal.tiers
      }
    }
  },
  methods: {
    edit() {
      this.loading = true
      this.$api.get('/clothing/' + this.item._id).then(res => {
        const update = {
          name: res.data.name,
          type: res.data.type,
          cultures: [
            ...new Set([...res.data.cultures, ...this.formData.cultures])
          ],
          tiers: [...new Set([...res.data.tiers, ...this.formData.tiers])]
        }

        const formData = new FormData()
        for (const prop in update)
          if (update.hasOwnProperty(prop)) formData.append(prop, update[prop])

        if (this.formData.flavor)
          formData.append('flavor', this.formData.flavor)
        if (this.formData.base) formData.append('base', this.formData.base)

        this.$api
          .put('/clothing/' + this.item._id, formData, {
            headers: { 'Content-Type': 'multipart/form-data' }
          })
          .then(res => {
            this.reset()
            this.$emit('close', 'success')
          })
          .catch(err => {
            console.error(err)
            this.reset()
            this.$emit('close', 'error')
          })
      })
    },
    reset() {
      this.loading = false
      this.$refs.flavor.clear()
      this.$refs.base.clear()
    }
  }
}
</script>