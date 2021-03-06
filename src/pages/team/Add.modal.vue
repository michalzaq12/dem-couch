<template>
    <v-stepper v-model="currentStep" :vertical="$vuetify.breakpoint.smAndUp" :alt-labels="$vuetify.breakpoint.xsOnly">
        <v-loading :active="isLoading" :is-full-page="true"/>
        <v-stepper-header v-if="$vuetify.breakpoint.xsOnly">
            <v-stepper-step :complete="currentStep > 1" step="1" color="demko">{{$t('global.teamName')}}</v-stepper-step>

            <v-divider></v-divider>

            <v-stepper-step :complete="currentStep > 2" step="2" color="demko">{{$t('global.discipline')}}</v-stepper-step>

            <v-divider></v-divider>

            <v-stepper-step :complete="currentStep > 3" step="3" color="demko">{{$t('global.location')}}</v-stepper-step>

            <v-divider></v-divider>

            <v-stepper-step step="4" color="demko">{{$t('global.thumbnail')}}</v-stepper-step>
        </v-stepper-header>

        <v-stepper-step v-if="$vuetify.breakpoint.smAndUp" :complete="currentStep > 1" step="1" color="demko">{{$t('global.teamName')}}</v-stepper-step>
        <v-stepper-content step="1">
            <v-form v-model="valid.name" @submit.prevent="valid.name ? currentStep = 2 : false">
                <div class="mb-5">
                    <div class="text-xs-center headline font-weight-bold">{{$t('addTeamForm.chooseTeam')}}</div>
                    <v-text-field color="demko" v-model="form.name" :label="'global.teamName' | translate" :rules="nameRules" required></v-text-field>
                </div>
                <div class="text-xs-center">
                    <v-btn flat @click="$emit('update:isOpen', false)">{{$t('button.cancel')}}</v-btn>
                    <v-btn :disabled="!valid.name" color="demko" class="white--text" @click="currentStep = 2">{{$t('button.continue')}}</v-btn>
                </div>
            </v-form>
        </v-stepper-content>

        <v-stepper-step v-if="$vuetify.breakpoint.smAndUp" :complete="currentStep > 2" step="2" color="demko">{{$t('global.discipline')}}</v-stepper-step>
        <v-stepper-content step="2">
            <v-form v-model="valid.discipline" @submit.prevent="currentStep = 3">
                <div class="mb-5">
                    <div class="text-xs-center headline font-weight-bold">{{$t('addTeamForm.chooseDiscipline')}}</div>
                    <v-radio-group v-model="form.discipline" :rules="disciplineRules" color="demko">
                        <v-radio color="demko" v-for="discipline in disciplines" :key="discipline.id" :label="discipline.name | translate" :value="discipline.value"></v-radio>
                    </v-radio-group>
                </div>
                <div class="text-xs-center">
                    <v-btn flat @click="currentStep = 1">{{$t('button.cancel')}}</v-btn>
                    <v-btn :disabled="!valid.discipline" color="demko" class="white--text" @click="currentStep = 3">{{$t('button.continue')}}</v-btn>
                </div>
            </v-form>
        </v-stepper-content>

        <v-stepper-step v-if="$vuetify.breakpoint.smAndUp" :complete="currentStep > 3" step="3" color="demko">{{$t('global.location')}}</v-stepper-step>
        <v-stepper-content step="3">

            <div class="mb-5">
                <div class="text-xs-center headline font-weight-bold">{{$t('addTeamForm.chooseLocation')}}</div>
                <autocomplete @place_changed="placeChanged"/>
            </div>
            <div class="text-xs-center">
                <v-btn flat @click="currentStep = 2">{{$t('button.cancel')}}</v-btn>
                <v-btn :disabled="!(form.location !== null && form.location.name !== null)" color="demko" class="white--text" @click="currentStep = 4">{{$t('button.continue')}}</v-btn>
            </div>

        </v-stepper-content>

        <v-stepper-step v-if="$vuetify.breakpoint.smAndUp" :complete="currentStep > 4" step="4" color="demko">{{$t('global.thumbnail')}}</v-stepper-step>
        <v-stepper-content step="4">
            <v-card class="mb-3 pb-5">
                <div class="pa-2 text-xs-center headline font-weight-bold">{{$t('addTeamForm.chooseThumbnail')}}</div>
                <div class="pa-2 text-xs-center subheading font-weight-normal">{{$t('global.canSkip')}}</div>
                <div class="pa-2 text-xs-center">

                    <v-layout align-center justify-space-between column class="upload-container">
                        <thumbnail-upload-button ref="upload">
                            {{$t('button.selectFile')}}
                            <v-icon right dark>cloud_upload</v-icon>
                        </thumbnail-upload-button>
                        <div class="elevation-1 thumb"><img v-if="$refs.upload && $refs.upload.files[0] && $refs.upload.files[0].thumb" :src="$refs.upload.files[0].thumb" width="auto" height="100" /></div>
                    </v-layout>

                </div>
            </v-card>
            <div class="text-xs-center">
                <v-btn flat @click="currentStep = 3">{{$t('button.cancel')}}</v-btn>
                <v-btn color="demko" class="white--text" @click="submit()">{{$t('button.continue')}}</v-btn>
            </div>

        </v-stepper-content>

    </v-stepper>
</template>



<script>
    import ThumbnailUploadButton from './ThumbUpload';
    import Autocomplete from '@/components/Autocomplete';

    import {mapActions}  from 'vuex';
    export default {
        name: 'add-team-modal',
        components: {
            ThumbnailUploadButton,
            Autocomplete
        },
        props: {
            isOpen: Boolean
        },
        data() {
            return {
                isLoading: false,
                form: {
                    name: null,
                    discipline: null,
                    thumbnail: null,
                    location: null
                },
                valid: {
                    name: false,
                    discipline: false
                },
                currentStep: 1,
                disciplines: [
                    {
                        id: 1,
                        name: 'discipline.football',
                        value: 'football',
                    },
                    {
                        id: 2,
                        name: 'discipline.basketball',
                        value: 'basketball'
                    },
                    {
                        id: 3,
                        name: 'discipline.volleyball',
                        value: 'volleyball'
                    },
                    {
                        id: 4,
                        name: 'discipline.handball',
                        value: 'handball'
                    }
                ],
                nameRules: [
                    v => !!v || this.$t('addTeamForm.required.teamName'),
                    v => (v && v.length > 6) || this.$t('validation.atLeast', {
                        count: '6'
                    })
                ],
                disciplineRules: [
                    v => !!v || this.$t('addTeamForm.required.teamDiscipline'),
                ]
            }
        },
        methods: {
            ...mapActions('userTeams', ['fetchTeams']),
            placeChanged(location) {
                this.form.location = location
            },
            edit(team){
                this.form = team;
                this.$emit('update:isOpen', true);
            },
            close(){
                this.reset(this.form);
                this.reset(this.valid, false);
                this.$refs.upload.reset();
                this.currentStep = 1;
                this.$emit('update:isOpen', false);
            },
            submit() {
                this.isLoading = true;
                this.$http.post('/teams', this.form).then(({data}) => {
                    return this.$refs.upload.send(`/teams/${data.id}/thumbnail`)
                }).then(() => {
                    this.fetchTeams();
                }).finally(() => {
                    this.close();
                    this.isLoading = false;
                })


            }
        }
    }
</script>
