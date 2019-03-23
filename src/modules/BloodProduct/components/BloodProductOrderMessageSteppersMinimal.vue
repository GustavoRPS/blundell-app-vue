<template>
  <v-stepper v-model="step">
      <v-stepper-header>
        <v-stepper-step :complete="step > 1" step="1">Paciente</v-stepper-step>
        <v-divider></v-divider>
        <v-stepper-step :complete="step > 2" step="2">Localização</v-stepper-step>
        <v-divider></v-divider>
        <v-stepper-step :complete="step > 3" step="3">Confirmação</v-stepper-step>
      </v-stepper-header>

      <v-stepper-items>
        <v-stepper-content step="1">
          <v-container fluid>
            <v-layout row align-center wrap> 
              <v-flex xs12 md6>
                <v-layout column align-center> 
                  <v-avatar
                    :size="200"
                    color="lighten-4"
                  >
                    <img :src="form.recipient_image.length ? form.recipient_image[0].url : 'https://www.gravatar.com/avatar/default?s=200&r=pg&d=mm'" alt="avatar">
                  </v-avatar>

                  <v-btn>
                    <file-upload
                      v-model="form.recipient_image"
                      ref="upload"
                      name="image"
                      extensions="gif,jpg,jpeg,png,webp"
                      accept="image/*"
                      post-action="/upload/post"
                      :drop="!edit"
                      @input-filter="inputFilter"
                      @input-file="inputFile"
                    >
                      Enviar Foto
                    </file-upload>
                  </v-btn>
                </v-layout>
              </v-flex>

              <v-flex xs12 md6>
                <v-text-field
                  v-model="form.recipient_name"
                  label="Nome do Paciente"
                  required
                />
                <v-radio-group
                  v-model="form.recipient_gender"
                >
                  <template v-slot:label>
                    Genero
                  </template>
                  <v-radio
                    color="blue"
                    label="Masculino" 
                    value="Male"
                  />
                  <v-radio 
                    color="red"
                    label="Feminino" 
                    value="Female"
                  />
                </v-radio-group>
                
                <v-select
                  v-model="form.recipient_bloodType"
                  :items="['A-','A+','B-','B+','AB-','AB+','O-','O+',]"
                  label="Tipo Sanguineo"
                ></v-select>
              </v-flex>
            </v-layout>
          </v-container>
        </v-stepper-content>

        <v-stepper-content step="2">
          <v-autocomplete
            v-model="data.place"
            :items="suggestedPlaces"
            :label="`Local de Doação`"
            prepend-icon="mdi-map-marker"
            item-text="name"
            item-value="formatted_address"
            auto-select-first
          >
          </v-autocomplete>
          
          <v-list three-line>
            <v-subheader>
              Sugestões de Locais {{data.place}}
            </v-subheader>
            <template v-for="(place, index) in suggestedPlaces">
              <v-list-tile 
                :key="place.id" 
                @click="selectSuggestedPlace(place)"
              >
                <v-list-tile-content>
                  <v-list-tile-title v-html="place.name"></v-list-tile-title>
                  <v-list-tile-sub-title v-html="place.formatted_address"></v-list-tile-sub-title>
                  <v-list-tile-sub-title v-html="place.international_phone_number"></v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile>
            </template>
          </v-list>
        </v-stepper-content>
      </v-stepper-items>

      <v-footer dark height="auto">
        <v-layout row justify-end>
          <v-btn 
            :disabled="step === 1"
            flat
            @click="goToStep(step-1)"
          >
            Voltar
          </v-btn>
          <v-btn
             :disabled="step === 3"
             color="primary"
             depressed
             @click="goToStep(step+1)"
           >
            Avançar
          </v-btn>

        </v-layout>
      </v-footer>
    </v-stepper>
</template>

<script>
  import FileUpload from 'vue-upload-component'

  import mock from './mock.json'

  let parsePlaceAddress = (place) => {
    let retriveComponentLongName = (name) => {
      return place.address_components.find((e) => { 
        return e.types.includes(name)
      }).long_name
    }

    return {
      location_name: place.name,
      location_address_street: retriveComponentLongName('route'),
      location_address_number: retriveComponentLongName('street_number'),
      location_address_district: retriveComponentLongName('locality'),
      location_address_locality: retriveComponentLongName('administrative_area_level_2'),
      location_address_region: retriveComponentLongName('administrative_area_level_1'),
      location_address_postal_code: retriveComponentLongName('postal_code'),
    }
  } 

  export default {
    components: {
      FileUpload,
    },
    data: () => ({
      edit: false,
      suggestedPlaces: mock.suggestedPlaces,
      step: 2,
      data: {
        place: null,
      },
      form: {
        recipient_image: [],
        recipient_name: null,
        recipient_gender: null,
        recipient_bloodType: null,
      }
    }),
    methods: {
      inputFile(newFile, oldFile, prevent) {
        if (newFile && !oldFile) {
          this.$nextTick(function () {
            this.edit = true
          })
        }
        if (!newFile && oldFile) {
          this.edit = false
        }
      },
      inputFilter(newFile, oldFile, prevent) {
        if (newFile && !oldFile) {
          if (!/\.(gif|jpg|jpeg|png|webp)$/i.test(newFile.name)) {
            this.alert('Your choice is not a picture')
            return prevent()
          }
        }
        if (newFile && (!oldFile || newFile.file !== oldFile.file)) {
          newFile.url = ''
          let URL = window.URL || window.webkitURL
          if (URL && URL.createObjectURL) {
            newFile.url = URL.createObjectURL(newFile.file)
          }
        }
      },
      selectSuggestedPlace(place) {
        // this.step++
        let parsedPlace = parsePlaceAddress(place)

        alert(JSON.stringify(parsedPlace))
      },
      goToStep(n) {
        this.step = n
        // TODO: move to watch
        this.$router.replace({query: {step:n}})
      }
    },
    watch: {
      'data.place': (newValue, oldValue) => {
        // alert(newValue)
      },
      edit(value) {
        if (value) {
          this.$nextTick(function () {
            if (!this.$refs.editImage) {
              return
            }
            let cropper = new Cropper(this.$refs.editImage, {
              aspectRatio: 1 / 1,
              viewMode: 1,
            })
            this.cropper = cropper
          })
        } else {
          if (this.cropper) {
            this.cropper.destroy()
            this.cropper = false
          }
        }
      }
    },
  }
</script>

<style>

</style>
