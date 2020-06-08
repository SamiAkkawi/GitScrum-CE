<script>
import Axios from '@utils/axios'
import Draggable from 'vuedraggable'
import InputEditable from '@components/utils/input-editable'
import ButtonLoading from '@components/utils/button-loading'
import { taskManager } from '@state/helpers'

export default {
  components: {
    Draggable,
    ButtonLoading,
    InputEditable,
  },
  props: {
    task: {
      type: Object,
      required: false,
      default: function() {
        return []
      },
    },
  },
  data() {
    return {
      checklists: [],
      loading: true,
      btnLoading: false,
      enabled: true,
      showEditParent: 0,
      progressbar: 0,
      showById: null,

      showChecklistForm: 0,
      checklistItem: '',
    }
  },
  computed: {
    ...taskManager,

    dragOptions() {
      return {
        animation: 0,
        group: 'description',
        disabled: false,
        ghostClass: 'ghost',
      }
    },
  },
  watch: {
    statusTask(data){
      if ( data.item.name === 'checklist.addList' ){
        this.checklists.push(data.item.object)
      }

      if ( data.item.name === 'checklist.reload' ){
        this.getChecklists()
      }
    },
  },
  created() {
    this.getChecklists()
  },
  methods: {
    getChecklists() {
      Axios()
        .get(
          'task-checklists/?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug +
            '&task_uuid=' +
            this.task.uuid
        )
        .then((response) => {
          this.checklists = response.data.data
          this.progressbar = response.data.stats.completed_percent
        })
    },

    addChecklistItem(index) {
      this.btnLoading = true
      this.alertStatus = false
      Axios()
        .post('task-checklists/?company_slug=' + 
        this.task.company.slug + 
        '&project_slug=' + 
        this.task.company.slug, 
        {
          parent_id: this.checklists[index].id,
          task_uuid: this.task.uuid,
          title: this.checklists[index].titleItem.trim(),
        })
        .then((response) => {
          this.toggleFormChecklist(index)
          this.getChecklists()
          this.btnLoading = false
        })
    },

    getProgressbarPercent(checklistItemId) {
      let total = 0
      let checked = 0

      for (let i = 0; i < this.checklists.length; i++) {
        total += this.checklists[i].children.length
        for (let j = 0; j < this.checklists[i].children.length; j++) {
          if (this.checklists[i].children[j].id === checklistItemId) {
            this.checklists[i].children[j].done.status = !this.checklists[i].children[j].done.status
          }
          if (this.checklists[i].children[j].done.status) {
            checked++
          }
        }
      }

      this.progressbar = parseFloat(((checked / total) * 100)).toFixed(0);
    },

    toggleChecklistItem(checklistItem) {
      let newStatus = checklistItem.done.status ? 'false' : 'true'
      this.getProgressbarPercent(checklistItem.id)
      Axios()
        .put(
          'task-checklists/' +
            checklistItem.id +
            '/toggle/?company_slug=' +
            this.task.company.slug +
            '&project_slug=' +
            this.task.project.slug +
            '&task_uuid=' +
            this.task.uuid,
          {
            status: newStatus,
          }
        )
    },
    removeChecklist(checklistId, checklist, title) {

     let message = this.$t('Do you really want to remove?') + ' "' + title + '"'

      this.$bvModal.msgBoxConfirm(message, this.msgBoxConfirmConfig() )
      .then(value => {
        
        if(value){
          this.loading = true

          Axios()
          .delete('task-checklists/' + checklistId)
          .then((response) => {
            let arr = []

            if (checklist !== '') {
              // In case of a checklistitem
              for (let i = 0; i < checklist.children.length; i++) {
                if (checklist.children[i].id !== checklistId) {
                  arr.push(checklist.children[i])
                }
              }
              checklist.children = arr
            } else {
              this.getChecklists()
            }
          })
          .catch((e) => {
            console.error(e)
          })
        }
      })

    },
    checklistMove(evt) {
      let id = null
      let newPosition = null

      if (evt.moved) {
        id = evt.moved.element.id
        newPosition = evt.moved.newIndex + 1
      }

      if (id && newPosition) {
        Axios()
          .put(
            'task-checklists/' +
              id +
              '/?company_slug=' +
              this.task.company.slug +
              '&project_slug=' +
              this.task.project.slug,
            {
              position: newPosition,
            }
          )
          .then((response) => {})
          .catch((e) => {})
      }
    },
    updateTitle(checklist){
      
      checklist.object.title = checklist.text

      Axios()
      .put(
        'task-checklists/' +
          checklist.object.id +
          '/?company_slug=' +
          this.task.company.slug +
          '&project_slug=' +
          this.task.project.slug +
          '&task_uuid=' +
          this.task.uuid,
        {
          title: checklist.text,
        }
      )

    },

    toggleFormChecklist(checklist) {
      this.showChecklistForm = checklist.id
    },
  },
}
</script>

<template>
  <b-row v-if="checklists[0]" class="mb-3 task-checklist">
    <b-col cols="1" class="task-left-icon">
      <font-awesome-icon 
      :icon="['far', 'list']" 
      class="task-icon" />
    </b-col>
    <b-col class="task-left-content">
      <div class="d-flex justify-content-between">
        <h5>{{ $t('Checklists') }}</h5>
        <span>{{ parseFloat(progressbar) | percent(0) }}</span>
      </div>
      <b-progress>
        <b-progress-bar 
        :style="{ 'background-color': task.workflow.color }" 
        :value="progressbar" 
        :max="100"></b-progress-bar>
      </b-progress>
      <div class="mt-3 wd-100">
        <Draggable 
        :list="checklists" 
        :disabled="!enabled" 
        group="list" 
        class="wd-100"
        v-bind="dragOptions" 
        @change="checklistMove">
          <div
          v-for="(checklist, index) in checklists" 
          :key="checklist.id" 
          class="mb-2 d-flex wd-100 task-checklist-item">
            <font-awesome-icon 
            v-if="authorize('tasks', 'update')"
            :icon="['fas', 'arrows-alt']"
            class="move-level-1 cursor-grab" />
          
            <div class="vlabeledit-label wd-100">

              <InputEditable v-if="authorize('tasks', 'update')" 
                :placeholder="$t('Task Title')"
                :text="checklist.title" 
                :current-object="checklist"
                class="font-weight-bold"
                @text-updated-blur="updateTitle"  
                @text-updated-enter="updateTitle"></InputEditable>
              <span v-else class="vlabeledit-label" v-text="checklist.title" />

              <span
                v-if="authorize('tasks', 'update')"
                class="checklist-remove"
                @click="removeChecklist(checklist.id, '', checklist.title)">
                <font-awesome-icon :icon="['fas', 'times']" style="color:#1E1E2F" />
              </span>

              <div v-if="checklist.user" class="checklist-details">
                {{ $t('Created by') }} <b-link :to="{
                  name: 'profile.user',
                  params: { username: checklist.user.username } }" 
                  v-text="checklist.user.name" /> 
                  {{ $t('at') }}
                <span v-text="checklist.created_at.date_for_humans" />
              </div>

                <Draggable
                  :group="{ name: `list-item-${checklist.id}` }"
                  :list="checklist.children"
                  v-bind="dragOptions"
                  @change="checklistMove">

                <div
                  v-for="checklistItem in checklist.children"
                  :key="checklistItem.id"
                  class="d-flex checklist-item-line">
                 
                  <font-awesome-icon 
                    v-if="authorize('tasks', 'update')" 
                    :icon="['fas', 'arrows-alt']" 
                    class="move-level-2 cursor-grab" />
                  <div class="wd-100">
                    <div class="d-flex wd-100">
                      <b-form-checkbox
                        v-if="showEditParent !== checklistItem.id"
                        :disabled="!authorize('tasks', 'update')"
                        :checked="checklistItem.done.status"
                        @change="toggleChecklistItem(checklistItem)"></b-form-checkbox>

                      <InputEditable v-if="authorize('tasks', 'update')" 
                        :placeholder="$t('Task Title')"
                        :text="checklistItem.title" 
                        :current-object="checklistItem"
                        @text-updated-blur="updateTitle"  
                        @text-updated-enter="updateTitle"></InputEditable>
                      <span v-else class="vlabeledit-label" v-text="checklistItem.title" />
                    </div>
                    <span
                      v-if="authorize('tasks', 'update')"
                      class="checklist-remove"
                      @click="removeChecklist(checklistItem.id, checklist, checklistItem.title)">
                      <font-awesome-icon :icon="['fas', 'times']" style="color:#9EA9C1" />
                    </span>
                    <div v-if="checklistItem.user" class="checklist-details">
                      {{ $t('Created by') }}
                      <b-link :to="{
                      name: 'profile.user',
                      params: { username: checklistItem.user.username } }" 
                      v-text="checklistItem.user.name" /> {{ $t('at') }} 
                      <span v-text="checklistItem.created_at.date_for_humans" />
                      <span v-if="checklistItem.done.user && checklistItem.done.status" class="d-block mt-1">
                        {{ $t('Checked by') }}
                        <b-link :to="{
                        name: 'profile.user',
                        params: { username: checklistItem.done.user.username } }" 
                        v-text="checklistItem.done.user.name" />
                      </span>
                    </div>
                  </div>
                </div>
              </Draggable>
              <div v-if="authorize('tasks', 'update')" class="new-checklist-item">

                <div v-if="showChecklistForm === checklist.id" class="mt-2">
                  <b-form-textarea
                    v-model="checklist.titleItem"
                    :placeholder="$t('Add checklist item')"
                    rows="3"
                    max-rows="60"
                    @keyup.enter="addChecklist"
                  ></b-form-textarea>
                  <b-link @click="toggleFormChecklist(index)">{{ $t('Cancel') }}</b-link>
                  <ButtonLoading
                    type="btn-md"
                    :loading="btnLoading"
                    :title="$t('add checklist item')"
                    :title-loading="$t('Loading')"
                    @action="addChecklistItem(index)"
                  ></ButtonLoading>
                </div>

                <div class="d-flex justify-content-end">
                  <b-link 
                  v-if="showChecklistForm !== checklist.id" 
                  @click="toggleFormChecklist(checklist)" 
                  v-text="$t('add checklist item')" />
                </div>
              </div>
            </div>
          </div>
          </Draggable>
      </div>
    </b-col>
  </b-row>
</template>
