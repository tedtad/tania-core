<template lang="pug">
  .upload-crop-task
    .modal-header
      span.h4.font-bold
        translate Watering
    .modal-body
      form(@submit.prevent="validateBeforeSubmit")
        .form-group
          label(for="type")
            translate Choose type of watering
          select.form-control#type(v-validate="'required'" :class="{'input': true, 'text-danger': errors.has('type') }" v-model="task.type" name="type" @change="typeChanged($event.target.value)")
            option(value="ALL")
              translate All
            option(value="PARTIAL")
              translate Partial
          span.help-block.text-danger(v-show="errors.has('type')") {{ errors.first('type') }}
        .form-group
          label
            translate Which crop do you want to water?
          .checkbox(v-for="crop in crops")
            label.i-checks
              input(type="checkbox" name="selected crops" v-validate="'required'" :class="{'input': true, 'text-danger': errors.has('selected crops') }" v-model="task.crops" v-bind:value="crop.uid")
              i
              | {{ crop.inventory.name }}
              .identifier-sm {{ crop.batch_id }}
          span.help-block.text-danger(v-show="errors.has('selected crops')") {{ errors.first('selected crops') }}
        .form-group
          button.btn.btn-success.pull-right(type="submit")
            i.fa.fa-check
            translate SAVE
          button.btn.btn-default(style="cursor: pointer;" @click="$parent.$emit('close')")
            i.fa.fa-close
            translate Cancel
</template>

<script>
import { mapGetters, mapActions } from 'vuex'
import { StubTask } from '../../../stores/stubs'
import moment from 'moment-timezone'
export default {
  name: "WaterTaskModal",
  created () {
    this.task.type = "PARTIAL"
    this.task.crops = []
  },
  data () {
    return {
      task: Object.assign({}, StubTask),
    }
  },
  methods: {
    ...mapActions([
      'waterCrop',
    ]),
    create () {
      this.task.source_area_id = this.area.uid
      this.task.watering_date = moment().format('YYYY-MM-DD HH:ss')
      for (var i = 0; i < this.task.crops.length; i++) {
        this.task.obj_uid = this.task.crops[i]
        this.waterCrop(this.task)
          .then(() => this.$parent.$emit('close'))
          .catch(() => this.$toasted.error('Error in water task submission'))
      }
    },
    typeChanged (type) {
      if (type === 'ALL') {
        for (var i = 0; i < this.crops.length; i++) {
          this.task.crops.push(this.crops[i].uid)
        }
      } else {
        this.task.crops = []
      }
    },
    validateBeforeSubmit () {
      this.$validator.validateAll().then(result => {
        if (result) {
          this.create()
        }
      })
    },
  },
  props: ['crops', 'area'],
}
</script>
