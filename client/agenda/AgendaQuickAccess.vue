<template lang="pug">
.agenda-quickaccess
  n-affix(:trigger-top='82')
    .card.shadow-sm
      .card-body
        n-button(
          id='agenda-quickaccess-filterbyareagroups-btn'
          block
          type='success'
          size='large'
          strong
          @click='agendaStore.$patch({ filterShown: true })'
          )
          i.bi.bi-funnel.me-2
          span {{ shortMode ? 'Filter...' : 'Filter Areas + Groups...' }}
          n-badge.ms-2(:value='agendaStore.selectedCatSubs.length', processing)
        n-button.mt-2(
          v-if='!agendaStore.pickerMode'
          id='agenda-quickaccess-picksessions-btn'
          block
          secondary
          type='success'
          size='large'
          strong
          @click='pickerStart'
          )
          i.bi.bi-ui-checks.me-2
          span {{ shortMode ? 'Pick...' : 'Pick Sessions...' }}
        .agenda-quickaccess-btnrow(v-else)
          .agenda-quickaccess-btnrow-title {{ shortMode ? 'Sess. Pick' : 'Session Selection' }}
          n-button.me-1(
            v-if='!agendaStore.pickerModeView'
            id='agenda-quickaccess-applypick-btn'
            type='success'
            size='large'
            strong
            @click='pickerApply'
            )
            i.bi.bi-check2-square.me-2
            span Apply
          n-button.me-1(
            v-else
            id='agenda-quickaccess-modifypick-btn'
            color='#6f42c1'
            size='large'
            strong
            @click='pickerModify'
            )
            i.bi.bi-pencil-square.me-2
            span Modify
          n-button.ms-1(
            id='agenda-quickaccess-discardpick-btn'
            secondary
            color='#666'
            size='large'
            strong
            @click='pickerDiscard'
            )
            i.bi.bi-x-square.me-2
            span Discard
        n-divider: small.text-muted Calendar
        n-button.mt-2(
          id='agenda-quickaccess-calview-btn'
          block
          color='#6c757d'
          size='large'
          strong
          @click='agendaStore.$patch({ calendarShown: true })'
          )
          i.bi.bi-calendar3.me-2
          span {{ shortMode ? 'Cal View' : 'Calendar View' }}
        n-dropdown(
          :options='downloadIcsOptions'
          size='large'
          :show-arrow='true'
          trigger='click'
          @select='downloadIcs'
          )
          n-button.mt-2(
            id='agenda-quickaccess-addtocal-btn'
            block
            secondary
            color='#6c757d'
            size='large'
            strong
            )
            i.bi.bi-calendar-check.me-2
            span {{ shortMode ? '.ics' : 'Add to your calendar...' }}
        template(v-if='agendaStore.meetingDays.length > 0')
          n-divider: small.text-muted Jump to...
          ul.nav.nav-pills.flex-column.small.agenda-quickaccess-jumpto
            li.nav-item(v-if='agendaStore.isMeetingLive')
              a.nav-link(
                href='#now'
                @click='scrollToNow'
                )
                i.bi.bi-arrow-right-short.d-none.d-xxl-inline.me-2
                span Now
            li.nav-item(v-for='day of agendaStore.meetingDays')
              a.nav-link(
                :class='agendaStore.dayIntersectId === day.slug ? `active` : ``'
                :href='`#slot-` + day.slug'
                @click='scrollToDay(day.slug, $event)'
                )
                i.bi.bi-arrow-right-short.d-none.d-xxl-inline.me-2
                span {{day.label}}
</template>

<script setup>
import { computed, h } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { DateTime } from 'luxon'
import {
  NAffix,
  NBadge,
  NButton,
  NDivider,
  NDropdown,
  useMessage
} from 'naive-ui'

import { useAgendaStore } from './store'
import { useSiteStore } from '../shared/store'
import { getUrl } from '../shared/urls'

// MESSAGE PROVIDER

const message = useMessage()

// STORES

const agendaStore = useAgendaStore()
const siteStore = useSiteStore()

// ROUTER

const router = useRouter()
const route = useRoute()

// Download Ics Options

const downloadIcsOptions = [
  {
    label: 'Subscribe... (webcal)',
    key: 'subscribe',
    icon: () => h('i', { class: 'bi bi-calendar-week text-blue' })
  },
  {
    label: 'Download... (.ics)',
    key: 'download',
    icon: () => h('i', { class: 'bi bi-arrow-down-square' })
  }
]

// COMPUTED

const shortMode = computed(() => {
  return siteStore.viewport <= 1350
})

// METHODS

function pickerStart () {
  agendaStore.$patch({ pickerMode: true })
}
function pickerApply () {
  agendaStore.$patch({ pickerModeView: true })
  agendaStore.persistMeetingPreferences()
}
function pickerModify () {
  agendaStore.$patch({ pickerModeView: false })
}
function pickerDiscard () {
  agendaStore.$patch({ pickerMode: false })
  if (route.query.show) {
    router.push({ query: null })
  }
}

function downloadIcs (key) {
  message.loading('Generating calendar file... Download will begin shortly.')
  let icsUrl = ''
  if (agendaStore.pickerMode) {
    const sessionKeywords = agendaStore.scheduleAdjusted.map(s => s.sessionKeyword)
    icsUrl = `${getUrl('meetingCalIcs', { meetingNumber: agendaStore.meeting.number })}?show=${sessionKeywords.join(',')}`
  } else if (agendaStore.selectedCatSubs.length > 0) {
    icsUrl = `${getUrl('meetingCalIcs', { meetingNumber: agendaStore.meeting.number })}?show=${agendaStore.selectedCatSubs.join(',')}`
  } else {
    icsUrl = `${getUrl('meetingCalIcs', { meetingNumber: agendaStore.meeting.number })}`
  }
  if (key === 'subscribe') {
    window.location.assign(`webcal://${window.location.host}${icsUrl}`)
  } else {
    window.location.assign(icsUrl)
  }
}

function scrollToDay (dayId, ev) {
  ev.preventDefault()
  document.getElementById(`agenda-day-${dayId}`)?.scrollIntoView(true)
}

function scrollToNow (ev) {
  ev.preventDefault()

  const lastEventId = agendaStore.findCurrentEventId()

  if (lastEventId) {
    document.getElementById(`agenda-rowid-${lastEventId}`)?.scrollIntoView(true)
  } else {
    message.warning('There is no event happening right now.')
  }
}

</script>

<style lang="scss">
.agenda-quickaccess {
  width: 300px;

  @media screen and (max-width: 1350px) {
    width: 150px !important;
  }

  .card {
    width: 300px;

    @media screen and (max-width: 1350px) {
      width: 150px;

      .card-body {
        padding: .5rem;
      }
    }
  }

  .card-body .n-button {
    overflow: hidden;
  }

  &-btnrow {
    border: 1px solid #CCC;
    padding: 8px 6px 6px 6px;
    border-radius: 5px;
    display: flex;
    justify-content: stretch;
    position: relative;
    text-align: center;
    margin-top: 12px;

    @media screen and (max-width: 1350px) {
      flex-direction: column;
    }

    &-title {
      position: absolute;
      top: -8px;
      font-size: 9px;
      font-weight: 600;
      color: #999;
      left: 50%;
      padding: 0 5px;
      background-color: #FFF;
      transform: translate(-50%, 0);
      text-transform: uppercase;
    }

    button {
      flex: 1;

      @media screen and (max-width: 1350px) {
        padding: 12px 0;
        margin-left: 0 !important;
        margin-right: 0 !important;

        & + button {
          margin-top: 6px;
        }
      }
    }
  }

  .n-divider {
    margin-top: 15px;
    margin-bottom: 15px;
  }
}
</style>
