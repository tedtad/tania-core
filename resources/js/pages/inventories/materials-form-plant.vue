<template lang="pug">
  .materials-create
    form(@submit.prevent="validateBeforeSubmit")
      .form-group
        label.control-label(for="name")
          translate Variety Name
        input.form-control#name(type="text" v-validate="'required'" :class="{'input': true, 'text-danger': errors.has('name') }" v-model="inventory.name" name="name")
        span.help-block.text-danger(v-show="errors.has('name')") {{ errors.first('name') }}
      .form-group
        .row
          .col-xs-6
            label.control-label(for="plant_type")
              translate Plant Type
            select.form-control#plant_type(v-validate="'required'" :class="{'input': true, 'text-danger': errors.has('plant_type') }" v-model="inventory.plant_type" name="plant_type")
              option(v-for="plant in options.plantTypes" v-bind:value="plant.key") {{ plant.label }}
            span.help-block.text-danger(v-show="errors.has('plant_type')") {{ errors.first('plant_type') }}
          .col-xs-6
            label.control-label
              translate Produced by
            input.form-control#produced_by(type="text" v-validate="'required'" :class="{'input': true, 'text-danger': errors.has('produced_by') }" v-model="inventory.produced_by" name="produced_by")
            span.help-block.text-danger(v-show="errors.has('produced_by')") {{ errors.first('produced_by') }}
      .form-group
        .row
          .col-xs-6
            label(for="quantity")
              translate Quantity
            input.form-control#quantity(type="text" v-validate="'required|decimal|min:0'" :class="{'input': true, 'text-danger': errors.has('quantity') }" v-model="inventory.quantity" name="quantity")
            span.help-block.text-danger(v-show="errors.has('quantity')") {{ errors.first('quantity') }}
          .col-xs-6
            label(for="quantity_unit")
              translate Unit
            select.form-control#quantity_unit(v-validate="'required'" :class="{'input': true, 'text-danger': errors.has('quantity_unit') }" v-model="inventory.quantity_unit" name="quantity_unit")
              option(v-for="unit in options.quantityUnits" v-bind:value="unit.key") {{ unit.label }}
            span.help-block.text-danger(v-show="errors.has('quantity_unit')") {{ errors.first('quantity_unit') }}
      .form-group
        .row
          .col-xs-6
            label(for="price_per_unit")
              translate Price per Unit
            .input-group.m-b
              span.input-group-addon
                translate &euro;
              input.form-control#price_per_unit(type="text" v-validate="'required'" :class="{'input': true, 'text-danger': errors.has('price_per_unit') }" v-model="inventory.price_per_unit" name="price_per_unit")
            span.help-block.text-danger(v-show="errors.has('price_per_unit')") {{ errors.first('price_per_unit') }}
          .col-xs-6
            label.control-label(for="expiration_date")
              translate Expiration date
            .input-group
              datepicker#expiration_date(type="text" v-validate="'required'" :class="{'input': true, 'text-danger': errors.has('expiration_date') }" v-model="inventory.expiration_date" name="expiration_date" input-class="form-control" ref="openCal")
              span.input-group-btn
                button.btn.btn-primary(type="button" v-on:click="openPicker")
                  i.fa.fa-calendar
              span.help-block.text-danger(v-show="errors.has('expiration_date')") {{ errors.first('expiration_date') }}
      .form-group
        label.control-label(for="notes")
          translate Additional Notes
        textarea.form-control#notes(type="text" :class="{'input': true, 'text-danger': errors.has('notes') }" v-model="inventory.notes" name="notes" rows="3")
        span.help-block.text-danger(v-show="errors.has('notes')") {{ errors.first('notes') }}
      .form-group
        button.btn.btn-addon.btn-success.pull-right(type="submit")
          i.fa.fa-plus
          translate Save
        button.btn.btn-default(type="button" style="cursor: pointer;" @click="closeModal()")
          translate Cancel
</template>

<script>
import { StubInventory } from '../../stores/stubs'
import { PlantTypes } from '../../stores/helpers/farms/plant'
import { PlantUnits } from '../../stores/helpers/inventories/inventory'
import { mapGetters, mapActions } from 'vuex'
import Datepicker from 'vuejs-datepicker';
import moment from 'moment';
export default {
  name: 'InventoriesMaterialsFormPlant',
  components: {
      Datepicker
  },
  data () {
    return {
      inventory: Object.assign({}, StubInventory),
      options: {
        plantTypes: Array.from(PlantTypes),
        quantityUnits: Array.from(PlantUnits),
      }
    }
  },
  methods: {
    ...mapActions([
      'submitMaterial',
      'openPicker',
    ]),
    submit () {
      this.inventory.expiration_date = moment(this.inventory.expiration_date).format('YYYY-MM-DD')
      this.inventory.type = "plant"
      this.submitMaterial(this.inventory)
        .then(() => this.$emit('closeModal'))
        .catch(() => this.$toasted.error('Error in material submission'))
    },
    closeModal () {
      this.$emit('closeModal')
    },
    openPicker () {
      this.$refs.openCal.showCalendar()
    },
    validateBeforeSubmit () {
      this.$validator.validateAll().then(result => {
        if (result) {
          this.submit()
        }
      })
    }
  },
  mounted () {
    if (typeof this.data.uid != "undefined") {
      this.inventory.uid = this.data.uid
      this.inventory.name = this.data.name
      this.inventory.plant_type = this.data.type.type_detail.plant_type.code
      this.inventory.produced_by = this.data.produced_by
      this.inventory.quantity = this.data.quantity.value
      this.inventory.quantity_unit = this.data.quantity.unit
      this.inventory.price_per_unit = this.data.price_per_unit.amount
      this.inventory.expiration_date = moment(this.data.expiration_date).format('YYYY-MM-DD')
      this.inventory.notes = this.data.notes
    } else {
      this.inventory.plant_type = this.options.plantTypes[0].key
      this.inventory.quantity_unit = this.options.quantityUnits[0].key
    }
  },
  props: ['data'],
}
</script>
