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
                    <img :src="data.avatar.length ? data.avatar[0].url : 'https://www.gravatar.com/avatar/default?s=200&r=pg&d=mm'" alt="avatar">
                  </v-avatar>

                  <v-btn>
                    <file-upload
                      v-model="data.avatar"
                      ref="upload"
                      name="image"
                      extensions="gif,jpg,jpeg,png,webp"
                      accept="image/*"
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
                  v-model="form.recipient_bloodtype"
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
            @change="selectPlace"
            prepend-icon="mdi-map-marker"
            item-text="name"
            item-value="address_components"
            auto-select-first
          >
          </v-autocomplete>
          
          <v-list three-line>
            <v-subheader>
              Sugestões de Locais
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

        <v-stepper-content step="3">
          <v-container fluid>
            <v-layout column align-center justify-center> 
              <v-flex v-if="loading">
                <v-progress-circular
                  :size="100"
                  color="red"
                  indeterminate
                ></v-progress-circular>
              </v-flex>

              <v-img v-if="!loading" :src="response.image" aspect-ratio="2.7" contain width="100%"/>
              
                <v-menu offset-y>
                  <template v-slot:activator="{ on }">
                    <v-btn
                      large
                      color="blue-grey"
                      class="white--text"
                      v-on="on"
                    >
                      Compartilhar
                    </v-btn>
                  </template>

                  <v-list>
                    <v-list-tile>
                      <v-list-tile-title>
                        Facebook
                      </v-list-tile-title>
                    </v-list-tile>
                  </v-list>
                </v-menu>
              <v-flex v-if="!loading">
              </v-flex>
            </v-layout>
          </v-container>
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
    props: {
      apiUrl: {
        type: String,
        default: `${process.env.VUE_APP_BLUNDELL_BLOOD_PRODUCT_ROOT_API}/api/v1/donations`,
      }
    },
    data: () => ({
      edit: false,
      suggestedPlaces: mock.suggestedPlaces,
      step: 3,
      response: {"image":"http:\/\/localhost:8000\/images\/banner-blood-donation-AalJelVUFVKd4g.png"},
      data: {
        place: null,
        avatar: [],
      },
      loading: false,
      form: {
        type: 'BloodDonation',
        recipient_image: null,
        recipient_name: null,
        recipient_gender: null,
        recipient_bloodtype: null,
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

        this.form.recipient_image = newFile.file
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
      selectPlace(address_components) {
        let parsedPlace = parsePlaceAddress({...{
          address_components: address_components
        }})
        this.form = {...this.form, ...parsedPlace}
        this.step++
      },
      selectSuggestedPlace(place) {
        let parsedPlace = parsePlaceAddress(place)
        this.form = {...this.form, ...parsedPlace}
        this.step++
      },
      goToStep(n) {
        this.step = n
        // TODO: move to watch
        this.$router.replace({query: {step:n}})
      }
    },
    watch: {
      'step': function(newValue, oldValue) {
        if (newValue === 3) {
          this.loading = true

          let formData = new FormData();
          Object.keys(this.form).forEach(key => formData.append(key, this.form[key]));
          
          let config = {
            headers: { 'content-type': 'multipart/form-data' }
          }

          this.$http.post(this.apiUrl, formData, config)
            .then((response) => {
              this.response = response.data
              this.loading = false
            })
            .catch(function (error) {
              throw error
            })
        }
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
