<template lang="pug">
  .reservoir-detail.col(v-if="loading === false")
    modal(v-if="showModal" @close="closeModal")
      FarmReservoirTaskForm(:data="reservoir" :asset="asset")
    .wrapper-md
      a#addTaskForm.btn.m-b-xs.btn-primary.btn-addon.pull-right(style="cursor: pointer;" @click="openModal()")
        i.fas.fa-plus
        translate Add Task
      h1.m-n.font-thin.h3.text-black {{ reservoir.name }}
    .wrapper
      .row.basicinfo
        .col-md-6
          .panel
            .panel-heading
              span.h4.text-lt
                translate Basic Info
            .panel-body
              .row.m-b
                .col-md-6
                  small.text-muted
                    translate Source Type
                  .h4.text-lt {{ getReservoirType(reservoir.water_source.type).label }}
                .col-md-6
                  small.text-muted
                    translate Capacity
                  .h4.text-lt {{ reservoir.water_source.capacity }}
              .row.m-b
                .col-md-6
                  small.text-muted
                    translate Used In
                  .h4.text-lt(v-for="area in reservoir.installed_to_area")
                    span.areatag {{ area.name }}
              .row.m-b
                .col-md-6
                  small.text-muted
                    translate Created On
                  .h4.text-lt {{ reservoir.created_date | moment('timezone', 'Asia/Jakarta').format('DD/MM/YYYY') }}
        .col-md-6
          .panel
            .panel-heading
              span.h4.text-lt
                translate Notes
            .panel-body
              form(@submit.prevent="validateBeforeSubmit")
                .input-group
                  input#content.form-control.input-sm(type="text" placeholder="Create a note" v-validate="'required'" :class="{'input': true, 'text-danger': errors.has('note.content') }" v-model="note.content" name="note.content")
                  span.input-group-btn
                    button.btn.btn-sm.btn-success(type="submit")
                      i.fa.fa-paper-plane
                  span.help-block.text-danger(v-show="errors.has('note.content')") {{ errors.first('note.content') }}
            ul.list-group.list-group-lg.no-bg.auto
              li.list-group-item.row(v-for="reservoirNote in reservoir.notes")
                .col-sm-9
                  span {{ reservoirNote.content }}
                  small.text-muted.clear.text-ellipsis {{ reservoirNote.created_date | moment('timezone', 'Asia/Jakarta').format('DD/MM/YYYY') }}
                .col-sm-3
                  button.btn.btn-xs.btn-default.pull-right(v-on:click="deleteNote(reservoirNote.uid)")
                    i.fa.fa-trash
      .row
        .task-list.col-xs-12
          .panel
            .panel-heading
              span.h4.text-lt
                translate Tasks
            TasksList(:domain="'RESERVOIR'" :asset_id="reservoir.uid" :reload="reload")
</template>

<script>
import { StubReservoir, StubNote } from '../../stores/stubs'
import { FindReservoirType } from '../../stores/helpers/farms/reservoir'
import { mapActions } from 'vuex'
import Modal from '../../components/modal.vue'
export default {
  name: 'Reservoir',
  data () {
    return {
      asset: 'Reservoir',
      loading: true,
      note: Object.assign({}, StubNote),
      reload: false,
      reservoir: Object.assign({}, StubReservoir),
      reservoirNotes: [],
      showModal: false,
    }
  },
  components: {
    FarmReservoirTaskForm: () => import('./tasks/task-form.vue'),
    TasksList: () => import('./tasks/task-list.vue'),
    Modal
  },
  created () {
    this.getReservoirByUid(this.$route.params.id)
      .then(({ data }) =>  {
        this.loading = false
        this.reservoir = data
      })
      .catch(error => console.log(error))
  },
  methods: {
    ...mapActions([
      'getReservoirByUid',
      'createReservoirNotes',
      'deleteReservoirNote',
    ]),
    closeModal () {
      this.showModal = false
      this.reload = !this.reload
    },
    create () {
      this.note.obj_uid = this.$route.params.id
      this.createReservoirNotes(this.note)
        .then(data => {
          this.reservoir = data
          this.note.content = ''
          this.$nextTick(() => this.$validator.reset())
        })
        .catch(({ data }) => this.message = data)
    },
    deleteNote(note_uid) {
      this.note.obj_uid = this.$route.params.id
      this.note.uid = note_uid
      this.deleteReservoirNote(this.note)
        .then(data => this.reservoir = data)
        .catch(({ data }) => this.message = data)
    },
    getReservoirType(key) {
      return FindReservoirType(key)
    },
    openModal() {
      this.data = {}
      this.showModal = true
    },
    validateBeforeSubmit () {
      this.$validator.validateAll().then(result => {
        if (result) {
          this.create()
        }
      })
    },
  }
}
</script>

