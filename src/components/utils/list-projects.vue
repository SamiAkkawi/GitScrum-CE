<script>

import Axios from '@utils/axios'
import ProjectVisibility from '@components/projects/project-visibility'
import ListLabels from '@components/utils/list-labels'
import Draggable from 'vuedraggable'
import ManageLabels from '@components/projects/modal/manage-label'
import moment from 'moment'

export default {
  components: {
    ProjectVisibility,
    ListLabels,
    Draggable,
    ManageLabels
  },
  props: {
    data: {
      type: Array,
      required: true,
    },
    disabled: {
      type: Boolean,
      required: true,
    },
    labels: {
      type: Boolean,
      required: true,
    },
  },
  data() {
    return {
      currentUser: JSON.parse(localStorage.getItem('CURRENT_USER')),
      currentCompany: JSON.parse(window.localStorage.getItem('CURRENT_COMPANY')),
      projectLabelSidebar: false,
      projectSelected: {},
      seriesTopSpark1: [
        {
          data: [],
        },
      ],
      chartOptionsTopSpark1: {
        chart: {
          sparkline: {
            enabled: true,
          },
        },
        colors: ['#606c87'],
        stroke: {
          curve: 'straight',
        },
        tooltip: {
          fixed: {
            enabled: false,
          },
          x: {
            show: false,
          },
          y: {
            title: {
              formatter: function(seriesName, object) {
                let date = moment()
                  .subtract(7 - object.dataPointIndex, 'day')
                  .format('YYYY-MM-DD')
                return '<small>[' + date + ']</small><br>Closed Tasks '
              },
            },
          },
          marker: {
            show: false,
          },
        },
        fill: {
          opacity: 0.2,
        },
        xaxis: {
          crosshairs: {
            width: 1,
          },
        },
        yaxis: {
          min: 0,
        },
      },
    }
  },
  methods: {
    seriesTopSpark(project) {
			return [
				{
					data: project.closed_tasks,
				},
			]
		},
		updatePosition(evt) {
			
			const projects = []
			const elements = document.getElementsByClassName('project-card')

			for (let i = 0; i < elements.length; i++) {
				projects.push(elements[i].getAttribute('data-slug'))
			}

			Axios()
				.put('/projects-positions/?company_slug=' + this.currentCompany.slug, {
					projects: projects,
				})
				.then((response) => {})
    },
		selectProject(project) {
      this.projectLabelSidebar = true;
			this.projectSelected = project
		},
		hideModal() {
			this.closeModal(this.$refs['modal'])
		},
		toggleModal() {
			this.$refs['modal'].toggle('#toggle-btn')
		},
		redirect(project) {
			this.$router.push({
				name: 'projects.board',
				params: {
					companySlug: project.company.slug,
					projectSlug: project.slug,
				},
			})
    },
    projectChangeStatus(project, code){
      Axios()
        .put('/projects/' + project.slug + 
          '/?company_slug=' + this.currentCompany.slug, {
					current_status: code,
				})
				.then((response) => {
          project.status.title = response.data.data.status.title
          project.status.code = code
        })
    },
    canProjectChangeStatus(project){
      return project.owner.username === this.currentUser.username
    }
  },
}
</script>

<template>
  <div>
    
    <b-sidebar
      v-model="projectLabelSidebar"
      no-header
      backdrop
      shadow
      right
      class="sidebar-project-labels"
    >
      <template v-slot="{ hide }">
        <div>
          <div class="p-2">
          <h5 id="sidebar-no-header-title">{{ projectSelected.name }}</h5>
          <p>{{ $t('Project Labels are used to help you organize projects.') }}</p>
          </div>
          <ManageLabels :project="projectSelected"></ManageLabels>
        </div>
      </template>
    </b-sidebar>
    
    <div class="d-flex justify-content-center">
      <Draggable
          :disabled="disabled"
          class="d-flex justify-content-center flex-wrap"
          ghost-class="ghost"
          draggable=".bunny-project-card"
          @end="updatePosition">


        <div v-for="(project,index) in data" :key="index"  class="bunny-project-card" :data-slug="project.slug">
          <div :id="project.slug" class="card">
            <div
              class="img cursor-grab"
              :style="
                'background: url(' +
                  project.background_thumb +
                  ') ;background-repeat: no-repeat;background-size: cover;'
              "
            >
              <img v-lazy="project.logo" class="logo" :alt="project.name" />
            </div>

            <b-progress
              class="project-list-progress"
              :value="project.percent"
              :max="100"
            ></b-progress>

            <div class="card-body">
            
                <router-link
                  :to="{
                    name: 'projects.board',
                    params: {
                      companySlug: project.company.slug,
                      projectSlug: project.slug,
                    },
                  }"
                  class="project-title txt-primary-title"
                >
                  {{ project.name }}
                </router-link>
              <div class="project-details d-flex justify-content-between">
                
                <div class="d-flex justify-content-start ">
                  
                  <ProjectVisibility
                    :visibility="project.visibility"
                  ></ProjectVisibility>
                  
                  <b-dropdown v-if="canProjectChangeStatus(project)" variant="link" class="styled-dropdown badge badge-light">
                    <template v-slot:button-content>
                        {{ project.status.title }}
                        &nbsp;<b>{{ parseFloat(project.percent) | percent(0) }}</b>
                        <font-awesome-icon :icon="['far', 'angle-down']" class="ml-2" />
                    </template>
                    <b-dropdown-item v-if="project.status.code !== 0" @click="projectChangeStatus(project, 0)">{{ $t('In Progress') }}</b-dropdown-item>
                    <b-dropdown-item v-if="project.status.code !== 1" @click="projectChangeStatus(project, 1)">{{ $t('Completed') }}</b-dropdown-item>
                    <b-dropdown-item v-if="project.status.code !== 2" @click="projectChangeStatus(project, 2)">{{ $t('Archived') }}</b-dropdown-item>
                  </b-dropdown>

                  <b-badge v-if="!canProjectChangeStatus(project)" variant="light">
                    {{ project.status.title }}
                    &nbsp;<b>{{ parseFloat(project.percent) | percent(0) }}</b>
                  </b-badge>

                  <b-button v-b-toggle="'collapse' + project.slug" size="sm">
                    <font-awesome-icon
                      :icon="['fa', 'chart-area']" />
                  </b-button>

                  <b-button size="sm" class="ml-1"  @click="selectProject(project)">
                    <font-awesome-icon :icon="['fa', 'tags']" />
                  </b-button>

                </div>
                
              </div>

              <b-collapse :id="'collapse' + project.slug">
                  <div class="small text-uppercase mt-2 pb-1">
                    <small class="small font-weight-bold">{{ $t('Project Progress in Last 7 Days') }}</small>
                  </div>
                  
                  <apexchart
                    type="area"
                    height="45"
                    :options="chartOptionsTopSpark1"
                    :series="seriesTopSpark(project)"
                    class="mt-1" />
              </b-collapse>

              <ListLabels
                v-show="project.labels[0] && labels"
                :labels="project.labels"
                class="mt-1 labels-size"></ListLabels>
            </div>
          </div>
        </div>
      </Draggable>
    </div>
  </div>
</template>