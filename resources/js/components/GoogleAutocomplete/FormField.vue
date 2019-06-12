<template>
  <default-field :field="field">
    <template slot="field">
      <div class="form-group">
        <vue-google-autocomplete
            ref="address"
            :id="field.name"
            class="w-full form-control form-input form-input-bordered"
            :class="errorClasses"
            :placeholder="placeholder"
            :country="field.countries"
            :types="field.type"
            v-model="value"
            v-on:keypress.enter.prevent=""
            v-on:placechanged="getAddressData">
        </vue-google-autocomplete>
      </div>

      <p v-if="value != ''" class="my-2 text-success">{{ translate.current_address }}: {{ value }}</p>

      <p v-if="hasError" class="my-2 text-danger">
        {{ firstError }}
      </p>
    </template>
  </default-field>
</template>

<script>
    import {FormField, HandlesValidationErrors} from 'laravel-nova'
    import VueGoogleAutocomplete from 'vue-google-autocomplete'

    export default {
        components: {VueGoogleAutocomplete},

        mixins: [FormField, HandlesValidationErrors],

        props: ['resourceName', 'resourceId', 'field'],

        data: function () {
            return {
                address: ''
            }
        },

        computed: {
            translate() {
                return Nova.config.google_autocomplete_translations
            },

            placeholder() {
                if (this.value != '') {
                    return this.translate.update_address
                }

                return this.field.name
            }
        },

        methods: {

            /**
             * Get address
             */
            getAddressData: function (addressData, placeResultData) {
                // Save current data address as a string
                this.handleChange(placeResultData.formatted_address)
                console.log(placeResultData)
                // Emmit events to by catch up for the other AddressMetadata fields
                this.field.addressObject.forEach(element => {
                    if (placeResultData.hasOwnProperty(element)) {
                        return this.emitAddressData(element, placeResultData[element])
                    }

                    if (!Object.keys(this.field.customValues).includes(element)) {
                        console.log("Yay");
                        placeResultData.address_components.forEach(component => {
                            if (component.types.includes(element)) {
                                console.log("emit", component)
                                return this.emitAddressData(element, component.long_name)
                            }
                        });
                    }

                    placeResultData.address_components.forEach(component => {
                        if (component.types.includes(element)) {
                            console.log("emit", component)
                            return this.emitAddressData(element, component.long_name)
                        }
                    });
                });
            },

            emitAddressData(key, value) {
                let event = `address-metadata-update-${key}`;
                console.log(event, value);
                Nova.$emit(event, {value: value})
            },

            /*
             * Set the initial, internal value for the field.
             */
            setInitialValue() {
                this.value = this.field.value || ''
            },

            /**
             * Fill the given FormData object with the field's internal value.
             */
            fill(formData) {
                formData.append(this.field.attribute, this.value || '')
            },

            /**
             * Update the field's internal value.
             */
            handleChange(value) {
                this.value = value
            }
        }
    }
</script>
