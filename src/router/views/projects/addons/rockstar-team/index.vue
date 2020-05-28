<script>
import Layout from '@layouts/tpl-main-project'
import Axios from '@utils/axios'
import DatePicker from 'vue2-datepicker'
import ListUsers from '@components/utils/list-users'
import TitleLoading from '@components/utils/title-loading'

export default {
  page: {
    title: 'Rockstar team',
    meta: [{ name: 'description', content: '' }],
  },
  components: { Layout, DatePicker, ListUsers, TitleLoading },
  data() {
    return {
      loading: true,
      dates: '',
      items: [],
      gridConfig: {
        crownColor: [
          '#FBD01E',
          '#C0C0C0',
          '#847E69',
          'transparent',
          'transparent',
          'transparent',
          'transparent',
          'transparent',
          'transparent',
          'transparent',
        ],
      },
      fields: [
        {
          key: 'user.avatar',
          label: 'Position',
        },
        {
          key: 'user.name',
          label: 'Team Member',
        },
        {
          key: 'total_efforts',
          label: 'Effort',
        },
        {
          key: 'closed_issues_count',
          label: 'Tasks',
        },
        {
          key: 'duration_worked',
          label: 'worked',
        }
      ],
    }
  },
  created() {
    this.getList('', '')
  },
  methods: {
    getList(dateStart, dateEnd) {
      Axios()
        .get(
          'rockstar-team/?company_slug=' +
            this.$route.params.companySlug +
            '&project_slug=' +
            this.$route.params.projectSlug +
            '&quantity=15&sort_by=tasks' +
            '&start=' +
            dateStart +
            '&end=' +
            dateEnd
        )
        .then((response) => {
          this.items = response.data.data
          this.loading = false
        })
        .catch((error) => {
          console.error(error)
        })
    },
    changeDate(date) {
      this.loading = true
      let d0 = new Date(date[0])
      let d1 = new Date(date[1])
      let dateStart = d0.getFullYear() + '-' + (d0.getMonth() + 1) + '-' + d0.getDate()
      let dateEnd = d1.getFullYear() + '-' + (d1.getMonth() + 1) + '-' + d1.getDate()
      this.getList(dateStart, dateEnd)
    },
  },
}
</script>

<template>
  <Layout>
    <template slot="header-left">
      <TitleLoading
        :title="$t('Rockstar Team')" :loading="loading"></TitleLoading>
    </template>

    <div slot="content" class="rockstar-team pt-10px">
      <div class="container">

        <div class="row mb-30-px">
          <div class="col-md-12 pr-0">
            
            <div class="mb-3">
              <DatePicker v-model="dates" range lang="en" type="date" confirm @change="changeDate"></DatePicker>
            </div>

            <b-table class="table-rockstar-team" striped hover :items="items" :fields="fields" >
              <template v-slot:cell(user.avatar)="data">
                <div class="d-flex align-items-center">
                  <span class="ranking-number">
                    {{data.index + 1 }}
                  </span>
                  <span class="ranking-crown" :style="'color:' + gridConfig.crownColor[data.index] + '!important'">
                    <font-awesome-icon :icon="['fas', 'crown']" />
                  </span>
                </div>
              </template>
              <template v-slot:cell(user.name)="data" >
                <div class="box-useravatar">
                  <ListUsers :user="data.item.user" :link="true" size="22"></ListUsers>
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
            </b-table>

          </div>
        </div>
      </div>
    </div>
  </Layout>
</template>
