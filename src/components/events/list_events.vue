<template>
  <div class="q-pa-md">
    <q-table
      style="min-height: 80vmin"
      row-key="name"
      :data="data"
      :columns="columns"
      :filter="filter"
      hide-header
      :loading="loading"
      grid
    >
      <template v-slot:top>
        <div class="col-6 q-table__title">{{title || 'Eventos'}}</div>

        <div class="col-6">

          <q-input class="float-right q-ml-lg" dense filled debounce="300" v-model="filter" placeholder="Search" style="max-width: 40%">
            <q-icon slot="append" name="search" />
          </q-input>

          <q-btn v-if="isManager" class="float-right" color="deep-purple-6" rounded no-caps label="Adicionar Eventos" @click="addEvents"/>
        </div>
      </template>

      <template v-slot:top-right>
        <q-input borderless dense debounce="300" v-model="filter" placeholder="Search">
          <q-icon slot="append" name="search" />
        </q-input>
      </template>

      <template v-slot:item="props">
        <div
          class="area-card"
        >
          <card-events
            :title="props.row.name"
            :description="props.row.description"
            :date="props.row.dateInitial"
            :isParticipateProp="props.row.isParticipating"
            :data="props.row"
            :isOnwer="isOnwer"
          />
        </div>
      </template>

      <template v-slot:no-data="{ icon, message, filter }">
        <div class="full-width row flex-center text-accent q-gutter-sm">
          <q-icon size="2em" name="sentiment_dissatisfied" />
          <span>
            Uhm que pena não temos eventos...
          </span>
          <q-icon size="2em" :name="filter ? 'filter_b_and_w' : icon" />
        </div>
      </template>

      <div slot="bottom" slot-scope="props" class="flex flex-center" style="width: 100%">
        <div class="q-pa-lg flex flex-center">
          <q-pagination
          v-model="props.pagination.page"
          :max="props.pagesNumber"
          :direction-links="true"
          >
          </q-pagination>
        </div>
      </div>

    </q-table>
  </div>
</template>
<script>
import CardEvents from './card_events.vue'

import { OPERATION } from 'src/enumerator/operation.js'
import { EventBus } from 'src/functions/event_bus.js'

export default {
  name: 'list-events',

  components: {
    'card-events': CardEvents
  },

  created () {
    EventBus.$on('on-save-event', (event) => {
      this.onShow = true
      this.event = event
      this.registerEvents(event)
    })

    EventBus.$on('on-delete-event', (event) => {
      this.deleteEvent(event)
    })

    EventBus.$on('on-leave-event', (event) => {
      this.leaveEvent(event)
    })
  },

  beforeDestroy () {
    EventBus.$off('on-save-event', this.event)
    EventBus.$off('on-delete-event')
    EventBus.$off('on-leave-event')
  },

  props: {
    title: {
      type: String
    },

    endpoint: {
      type: String
    },

    isOnwer: {
      type: Boolean,
      default: false
    },

    isManager: {
      type: Boolean,
      default: false
    }
  },

  data () {
    return {
      filter: '',
      loading: false,
      columns: [
        {
          name: 'name',
          required: true,
          label: 'Titulo',
          align: 'left',
          field: row => row.name,
          format: val => `${val}`,
          sortable: true
        },
        { name: 'description', align: 'center', label: 'description', field: 'description', sortable: true },
        { name: 'isParticipating', align: 'center', label: 'isParticipating', field: 'isParticipating', sortable: true }
      ],
      data: []
    }
  },

  mounted () {
    this.getEvents()
  },

  methods: {
    refresh () {
      this.getEvents()
    },

    addEvents () {
      EventBus.$emit('on-create-event')
    },

    saveEvents () {
      const value = {
        operation: OPERATION.create
      }
      this.registerEvents(value)
    },

    async getEvents () {
      try {
        this.loading = true
        this.data = []
        const result = await this.$axios.get(this.endpoint, this.dataEndpoint)

        this.data = result.data

        this.loading = false
      } catch (e) {
        this.loading = false
        console.log(e)
      }
    },

    async registerEvents (event) {
      try {
        let axiosFunction = this.$axios.post
        let url = '/event/'

        if (event.id) {
          url += `${event.id}/`
          axiosFunction = this.$axios.put
        }

        this.loading = true

        await axiosFunction(url, event)

        this.getEvents()

        this.loading = false
      } catch (e) {
        console.log(e)
        this.loading = false
      }
    },

    async leaveEvent (event) {
      try {
        this.loading = true

        await this.$axios.post(`userevent/leaveevent/${event.id}?userId=${localStorage.getItem('userId')}`)
        this.loading = false
      } catch (e) {
        console.log(e)
        this.loading = false
      }
    },

    async deleteEvent (event) {
      try {
        this.loading = true
        await this.$axios.delete(`/event/${event.id}`)
        this.getEvents()
        this.loading = false
      } catch (e) {
        this.loading = false
        console.log(e)
      }
    }
  }
}
</script>

<style lang="stylus">
.area-card
  margin: 2rem
  border-radius: 15px
  width: 20%
  height 216px
  transition .3s
  box-shadow: 0 1px 2px rgba(0,0,0,0.12), 0 1px 1px rgba(0,0,0,0.24);

.area-card:hover
  box-shadow: 0 14px 28px rgba(0,0,0,0.25), 0 10px 10px rgba(0,0,0,0.22);
  transform: translate(2px, -2px)

@media (max-width: 1440px)
  .area-card
    width: 30%;

</style>
