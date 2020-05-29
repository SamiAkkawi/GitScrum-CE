<script>
import Layout from '@layouts/tpl-main-project'
import Axios from '@utils/axios'
import DatePicker from 'vue2-datepicker'
import ListUsers from '@components/utils/list-users'
import TitleLoading from '@components/utils/title-loading'
import ButtonLoading from '@components/utils/button-loading'
import moment from 'moment'
import { modalManager } from '@state/helpers'

export default {
  page: {
    title: 'Time Tracking',
    meta: [{ name: '', content: '' }],
  },
  components: {
    Layout,
    DatePicker,
    ListUsers,
    TitleLoading,
    ButtonLoading
  },
  data() {
    return {
      loading: false,
      btnLoading: false,
      totalRows: 0,
      totalPages: 1,
      perPage: 15,
      currentPage: this.$route.query.page ? this.$route.query.page : 1,
      
      dateStart: '',
      dateEnd: '',
      filterTitle: '',
      filterMembers: [],
      optionMembers: [],
      dates: [],
      items: [],
      stats: [],
      resume: [],
      
      fields: [
        {
          key: 'user.name',
          label: 'Time Tracking',
        },
        {
          key: 'time.total',
          label: 'Worked',
        }
      ],
    }
  },
  created() {
    this.getTimeTracking(this.currentPage, '', '')
  },
  methods: {
    ...modalManager,

    modal(value, task) {
      this.open({ name: value, object: task })
    },

    scrollToTop() {
      window.scroll({
        top: 0,
        left: 0,
        behavior: 'smooth',
      })
    },

    setProjectMembers(){

      this.optionMembers = []
      let project = this.$store.getters['project/getProject']

      for (let i = 0; i < project.users.length; i++) {
        this.optionMembers.push(project.users[i].name)
      }

    },

    getTimeTracking(page, dateStart, dateEnd) {
      this.scrollToTop()
      this.loading = true
      Axios()
        .get(
          'time-trackings/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug +
            '&start=' +
            dateStart +
            '&end=' +
            dateEnd +
            '&users=' + 
            this.filterMembers +
            '&page=' +
            page
        )
        .then((response) => {
          if (response.data.data.data.length === 0 && page !== 1) {
            this.getTimeTracking(1, '', '')
            return
          }
          this.items = response.data.data.data
          this.totalRows = response.data.total
          this.totalPages = response.data.total_pages
          this.perPage = response.data.per_page
          this.currentPage = response.data.current_page
          this.resume = response.data.data.resume

          this.loading = false
          this.setProjectMembers()

          this.series = [
            {
              name: this.$t('Closed tasks'),
              data: response.data.data.stats,
            },
          ]
        })
    },

    changeDate(date) {

      this.dateStart = ''
      this.dateEnd = ''

      if (date[0]){
        let d0 = new Date(date[0])
        let dateStart = d0.getFullYear() + '-' + (d0.getMonth() + 1) + '-' + d0.getDate() + ' 00:00'
        this.dateStart = moment.utc(dateStart).format('YYYY-MM-DD HH:mm:ss')
      }

      if (date[1]){
        let d1 = new Date(date[1])
        let dateEnd = d1.getFullYear() + '-' + (d1.getMonth() + 1) + '-' + d1.getDate() + ' 23:59'
        this.dateEnd = moment.utc(dateEnd).format('YYYY-MM-DD HH:mm:ss')
      }

    },

    periodStart() {
      return this.resume.period_start ? moment.utc(this.resume.period_start).format('YYYY-MM-DD HH:mm:ss') : '-'
    },

    periodEnd() {
      return this.resume.period_end ? moment.utc(this.resume.period_end).format('YYYY-MM-DD HH:mm:ss') : '-'
    },

    periodHours() {
      return this.resume.worked_hours ? this.resume.worked_hours : '-'
    },

    downloadExcel() {
      Axios()
        .get(
          '/export-time-trackings/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug +
            '&users=' + 
            this.filterMembers +
            '&start_date=' +
            this.resume.period_start +
            '&end_date=' +
            this.resume.period_end,
          { responseType: 'blob' }
        )
        .then((response) => {
          const url = window.URL.createObjectURL(new Blob([response.data]))
          const link = document.createElement('a')
          link.href = url
          link.setAttribute('download', 'time-tracking.xlsx')
          document.body.appendChild(link)
          link.click()
        })
    },

    removeTime(id) {

      this.$bvModal.msgBoxConfirm(this.$t('Are you sure you want to delete this time tracking?'), this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){

          Axios()
          .delete(
          '/time-trackings/' +
            id +
            '/' +
            '?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug
          )
          .then((response) => {
            let found = this.items.find((t) => t.time.id === id)
            this.items.splice(this.items.indexOf(found), 1)
          })
          .catch((error) => {
            this.alertMessage = 'Error. ' + error.response.data.message
            this.alertStatus = true
          })

        }
      })
      
    },

    addFilter(){
      this.getTimeTracking(1, this.dateStart, this.dateEnd)
    }
  },
}
</script>

<template>
  <Layout>
    <template slot="header-left">
      <TitleLoading
        :title="$t('Time Tracking')" :loading="loading"></TitleLoading>
    </template>

    <div slot="content" class="time-stracking pt-10px">

      <div class="container">
        
        <div class="card mb-2">
          <div class="card-body">
            <div class="d-flex justify-content-between align-items-center">
              <div class="d-flex justify-content-start">
                <div class="media border-right mr-4">
                  <div class="media-body mr-4">
                    <label class="mb-0 fw-700">{{ $t('Hours Worked') }}</label>
                    <p class="mb-0" v-text="periodHours()"></p>
                  </div>
                </div>
                <div class="media">
                  <div class="media-body mr-4">
                    <label class="mb-0 fw-700">{{ $t('Start Date') }}</label>
                    <p class="mb-0" v-text="periodStart()"></p>
                  </div>
                </div>
                <div class="media">
                  <div class="media-body mr-4">
                    <label class="mb-0 fw-700">{{ $t('Due Date') }}</label>
                    <p class="mb-0" v-text="periodEnd()"></p>
                  </div>
                </div>
              </div>
              <button class="btn btn-secondary btn-sm" @click="downloadExcel">
                {{ $t('Export to CSV') }}
              </button>
            </div>
          </div>
        </div>

        <div class="d-flex justify-content-start mb-2">
          <b-form-tags v-model="filterMembers" size="sm" add-on-change no-outer-focus class="mb-2">
            <template v-slot="{ tags, inputAttrs, inputHandlers, disabled, removeTag }">
              <b-form-select
                :options="optionMembers"
                size="sm"
                v-bind="inputAttrs"
                v-on="inputHandlers"
                >
                <template v-slot:first>
                  <option disabled value="">{{ $t('Filter by team member') }}</option>
                </template>
              </b-form-select>
              <ul v-if="tags.length > 0" class="list-inline d-inline-block mb-2">
                <li v-for="tag in tags" :key="tag" class="list-inline-item">
                  <b-form-tag
                    :title="tag"
                    :disabled="disabled"
                    @remove="removeTag(tag)"
                  >{{ tag }}</b-form-tag>
                </li>
              </ul>
            </template>
          </b-form-tags>

          <DatePicker
            v-model="dates"
            range
            lang="en"
            type="date"
            confirm
            :not-after="new Date()"
            @change="changeDate"></DatePicker>

          <b-form-input 
            v-model="filterTitle" 
            type="search" 
            autocomplete="off" 
            size="sm"
            :placeholder="$t('Search by title')"></b-form-input>

          <div>
            <ButtonLoading
              :loading="btnLoading"
              type="btn-sm"
              icon="search"
              @action="addFilter"
            ></ButtonLoading>
          </div>

        </div>

        <b-table class="table-time-tracking" striped hover :items="items" :fields="fields" >
          <template v-slot:cell(user.name)="data" >
            
            <b-link v-if="data.item.task.code" href="#" class="txt-primary-title" @click="modal('task', data.item.task)">{{ data.item.task.code }} - {{ data.item.task.title }}</b-link >
            <b-link v-if="!data.item.task.code" href="#" class="txt-primary-title" @click="modal('task', data.item.task)">{{ data.item.task.title }}</b-link>

            <div class="date-time">
              <span v-text="data.item.time.start.timezone"></span>
                - 
              <span v-text="data.item.time.end.timezone"></span>
               <div class="time-comment" v-text="data.item.comment"></div>
            </div>

            <div class="box-useravatar">
              <router-link
              :to="{
                name: 'profile.user',
                params: { username: data.item.user.username },
              }"
              >
                {{data.item.user.name}}
              </router-link>
            </div>
          </template>
          <template v-slot:cell(time.total)="data" >
            <div class="worked">
                <span v-text="data.item.time.total"></span>
            </div>
            <a v-if="authorize('tasks', 'delete')" 
              href="javascript:;"
              @click="removeTime(data.item.time.id)"
            > {{ $t('Delete') }} </a>
          </template>
        </b-table>

        <div v-if="totalPages > 1" class="d-flex justify-content-center mg-b-30">
          <b-pagination
            hide-goto-end-buttons
            class="paginator"
            v-model="currentPage"
            :total-rows="totalRows"
            :per-page="perPage"
            @change="getTimeTracking($event, '', '')"
          >
            <template slot="prev-text">
              <font-awesome-icon :icon="['far', 'angle-left']" style="font-size:18px; color: #909CB8;" />
            </template>
            <template slot="next-text">
              <font-awesome-icon :icon="['far', 'angle-right']" style="font-size:18px; color: #909CB8;" />
            </template>
          </b-pagination>
        </div>

      </div>

      




      




      <div class="container">
        <div class="row mb-30-px">
          <div class="col-md-12">
            
            
            

            



            

            

            

  <!--
            <div class="divTable">
              <div class="divTableHead">
                <div class="divTableRow">
                  <div class="divTableCell" :style="gridConfig.style[0]"> </div>
                  <div class="divTableCell text-left" :style="gridConfig.style[1]">
                    {{ $tc('Task', 1) }}
                  </div>
                  <div class="divTableCell text-right" :style="gridConfig.style[4]">
                    {{ $t('Worked') }}
                  </div>
                </div>
              </div>
              <div class="divTableBody card">
                <div v-for="(item, index) in items" :key="index" class="divTableRowBg">
                  <div class="divTableRow">
                    <div class="divTableCell text-center" :style="gridConfig.style[0]">
                      <ListUsers :link="true" :user="item.user"> </ListUsers>
                    </div>

                    <div class="divTableCell text-left rockstar-king" :style="gridConfig.style[1]">
                      
                      <span class="d-block info">
                        <span v-text="item.time.start.timezone"></span>
                        /
                        <span v-text="item.time.end.timezone"></span>
                      </span>
                    </div>
                    <div class="divTableCell text-right rockstar-king" :style="gridConfig.style[2]">
                      <span style="    padding: 0 0 10px 18px;" class="ranking-number text-right d-block tx-20px fw-700 text-primary">
                        {{ item.time.total }}
                      </span>
                      <div
                      v-if="authorize('tasks', 'delete')"
                      class="card-delete"
                      @click="removeTime(item.time.id)"
                    >
                      {{ $t('Delete') }}
                    </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <div class="mg-b-20 mg-t-20">
              <div class="table-responsive">
                <table class="table">
                  <thead>
                    <tr>
                      <th width="10"></th>
                      <th></th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr>
                      <td class="align-middle"> </td>
                      <td class="align-middle">
                        <div class="mt-1"> </div>
                      </td>
                      <td class="align-middle text-right"> </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div> 
            -->


          </div>
        </div>
      </div>
    </div>
  </Layout>
</template>
