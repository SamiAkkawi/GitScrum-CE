<script>

import Axios from '@utils/axios'
import ProjectVisibility from '@components/projects/project-visibility'
import ListLabels from '@components/utils/list-labels'
import moment from 'moment'
import Draggable from 'vuedraggable'
import ManageLabels from '@components/projects/modal/manage-label'

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
  },
  data() {
    return {
      currentUser: JSON.parse(localStorage.getItem('CURRENT_USER')),
      currentCompany: JSON.parse(window.localStorage.getItem('CURRENT_COMPANY')),
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
    moment() {
			return moment()
		},
		
    seriesTopSpark(project) {
			return [
				{
					data: project.closed_tasks,
				},
			]
		},
		updatePosition(evt) {
			$(evt.item).removeClass('move-card')
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
		openLabelsManagementModal(project) {
			this.projectSelected = project
			this.$refs['modal'].show()
		},
		hideModal() {
			this.closeModal(this.$refs['modal'])
		},
		toggleModal() {
			this.$refs['modal'].toggle('#toggle-btn')
		},

		updateClass(event) {
			$(event.item).addClass('move-card')
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
    
    <b-modal id="modal" ref="modal" class="modal-manage-labels" hide-footer lazy size="xl">
      <div slot="modal-title">
        <span class="title">{{ projectSelected.name }}</span>
        <span class="subTitle">{{ $t('Manage Labels') }}</span>
      </div>
      <div class="modalContent">
        <ManageLabels :project="projectSelected"></ManageLabels>
      </div>
    </b-modal>
    
    <div class="d-flex justify-content-center">
      <Draggable
          :disabled="disabled"
          class="d-flex justify-content-center flex-wrap"
          ghost-class="ghost"
          draggable=".bunny-project-card"
          @start="updateClass"
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
                  class="project-title"
                >
                  {{ project.name }}
                </router-link>
              <div class="project-details d-flex justify-content-between">
                
                <div class="d-flex justify-content-start ">
                  
                  <ProjectVisibility
                    :visibility="project.visibility"
                  ></ProjectVisibility>
                  
                  <b-dropdown v-if="canProjectChangeStatus(project)" variant="link" class="styled-dropdown">
                    <template v-slot:button-content>
                      <b-badge variant="light">
                        {{ project.status.title }}
                        &nbsp;<b>{{ parseFloat(project.percent) | percent(0) }}</b>
                        <font-awesome-icon :icon="['far', 'angle-down']" class="ml-2" />
                      </b-badge>
                    </template>
                    <b-dropdown-item v-if="project.status.code !== 0" @click="projectChangeStatus(project, 0)">In Progress</b-dropdown-item>
                    <b-dropdown-item v-if="project.status.code !== 1" @click="projectChangeStatus(project, 1)">Completed</b-dropdown-item>
                    <b-dropdown-item v-if="project.status.code !== 2" @click="projectChangeStatus(project, 2)">Archived</b-dropdown-item>
                  </b-dropdown>

                  <b-badge v-if="!canProjectChangeStatus(project)" variant="light">
                    {{ project.status.title }}
                    &nbsp;<b>{{ parseFloat(project.percent) | percent(0) }}</b>
                  </b-badge>

                  <b-button v-b-toggle="'collapse' + project.slug" size="sm">
                    <font-awesome-icon
                      :icon="['fa', 'chart-area']" />
                  </b-button>

                </div>

                <div class="d-flex">
                  <a
                    class="project-text collapsed mr-8-px"
                    href="javascript:;"
                    aria-expanded="false"
                    @click="openLabelsManagementModal(project)"
                  >
                    <font-awesome-icon
                      :icon="['fa', 'tags']"
                      style="font-size: 14px;"
                    />
                  </a>
                  
                   
                </div>
                
              </div>

              <b-collapse :id="'collapse' + project.slug">
                <b-card>
                  <div class="title mt-1">
                    {{ $t('Project Progress in Last 7 Days') }}
                  </div>
                  
                  <apexchart
                    type="area"
                    height="45"
                    :options="chartOptionsTopSpark1"
                    :series="seriesTopSpark(project)"
                    class="mt-1" />
                </b-card>
              </b-collapse>

              <ListLabels
                v-show="project.labels[0]"
                :labels="project.labels"
                class="mt-1 labels-size"></ListLabels>
            </div>
          </div>
        </div>
      </Draggable>
    </div>
  </div>
</template>