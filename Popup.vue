<template>
    <div class="telephony" v-if="open" :class="open? 'telephony_active':''">
        <div class="telephony__cover">
            <div class="telephony__container">
                <div class="telephony__header">
                    <svg class="telephony__header-close" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 341.333 341.333" @click="close"><g xmlns="http://www.w3.org/2000/svg" ref="ppWindowCloseSVGG"><polygon points="341.333,149.333 81.707,149.333 200.853,30.187 170.667,0 0,170.667 170.667,341.333 200.853,311.147 81.707,192     341.333,192   "/></g></svg>
                    <div class="telephony__header-text" v-text="clientName"></div>
                </div>
                <div class="telephony__nav" :class="{'telephony__nav_triple': !clientExist}">
                    <template v-if="clientExist">
                        <span v-text="$t('telephony.popup.removeDialog')" class="telephony__nav-remove" @click="removeDialog"></span>
                        <span v-text="$t('telephony.popup.toCard')" class="telephony__nav-to-card" @click="openClient"></span>
                    </template>
                    <template v-else>
                        <span v-text="$t('telephony.popup.createClient')" @click="openCreateClient"></span>
                        <span v-text="$t('telephony.popup.addToClient')" @click="addToClient"></span>
                        <span v-text="$t('telephony.popup.removeDialog')" @click="removeDialog"></span>
                    </template>
                </div>
                <div class="telephony__history">
                    <div class="telephony__item" v-for="item in history" :key="item.id" @click.prevent="openCallActions(item)" @contextmenu.prevent="openCallActions(item)">
                        <div v-text="itemDate(item.created)"></div>
                        <div v-text="$t('telephony.types.'+item.type)"></div>
                        <div v-text="$t('telephony.status.'+item.status)" :class="{'telephony__item-not-work': item.status === 'not_worked'}"></div>
                        <div v-text="employee"></div>
                        <div v-text="callDate(item)"></div>
                    </div>
                </div>
                <div class="telephony__filters">
                    <div :class="{'telephony__filter-active': filters.type}">
                        <span>
                            {{ $t('telephony.types.title') }}
                            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M7.41,8.58L12,13.17L16.59,8.58L18,10L12,16L6,10L7.41,8.58Z"/></svg>
                        </span>
                        <select v-model="filters.type">
                            <option disabled>{{ $t('clients.table.status.first') }}</option>
                            <option v-for="item in filtersList.type" :key="item" :value="item">{{ $t('telephony.types.' + item) }}</option>
                            <option :value="null">{{ $t('clients.table.status.all') }}</option>
                        </select>
                    </div>
                    <div :class="{'telephony__filter-active': filters.status}">
                        <span>
                            {{ $t('telephony.status.title') }}
                            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M7.41,8.58L12,13.17L16.59,8.58L18,10L12,16L6,10L7.41,8.58Z"/></svg>
                        </span>
                        <select v-model="filters.status">
                            <option disabled>{{ $t('clients.table.status.first') }}</option>
                            <option v-for="item in filtersList.status" :key="item" :value="item">{{ $t('telephony.status.' + item) }}</option>
                            <option :value="null">{{ $t('clients.table.status.all') }}</option>
                        </select>
                    </div>
                    <div>
                        <span v-text="employee"></span>
                    </div>
                </div>

                <div class="telephony__actions" v-if="opendActionCall" @click="closeCallActions" ref="telephonyCallActions">
                    <div class="telephony__actions-cover">
                        <div v-text="$t('telephony.actions.changeStatus', {status: $t('telephony.status.'+((opendActionCall.status === 'worked')? 'not_worked':'worked')) })" @click="changeStatus"></div>
                        <div v-text="$t('telephony.actions.play')" :class="{'inactive': !opendActionCall.record}" @click="openVoiceRecord"></div>
                        <div class="inactive" v-text="$t('telephony.actions.call')"></div>
                        <div v-text="$t('telephony.actions.remove')" @click="removeDialog"></div>
                    </div>
                </div>

                <div class="telephony__record" v-if="opendVoiceRecord" @click="closeVoiceRecord" ref="telephonyVoiceRecord">
                    <div class="telephony__record-cover">
                        <span class="telephony__record-close" @click="closeVoiceRecord('clean')"><svg viewBox="0 0 24 24"><path fill="currentColor" d="M19,6.41L17.59,5L12,10.59L6.41,5L5,6.41L10.59,12L5,17.59L6.41,19L12,13.41L17.59,19L19,17.59L13.41,12L19,6.41Z" /></svg></span>
                        <audio controls :src="opendVoiceRecord.record">
                            Your browser does not support the
                            <code>audio</code> element.
                        </audio>
                    </div>
                </div>

                <div class="telephony__add-to-client" v-if="addToClientWindow" @click="closeAddToClient" ref="telephonyAddToClient">
                    <div class="telephony__add-to-client-cover">
                        <div class="telephony__add-to-client-title">
                            <svg xmlns="http://www.w3.org/2000/svg" @click="closeAddToClient(null)" viewBox="0 0 341.333 341.333" class="telephony__header-close"><g xmlns="http://www.w3.org/2000/svg"><polygon points="341.333,149.333 81.707,149.333 200.853,30.187 170.667,0 0,170.667 170.667,341.333 200.853,311.147 81.707,192     341.333,192   "></polygon></g></svg>
                            <span v-text="$t('telephony.popup.addToClient')"></span>
                        </div>
                        <div class="telephony__add-to-client-select">
                            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M7.41,8.58L12,13.17L16.59,8.58L18,10L12,16L6,10L7.41,8.58Z"/></svg>
                            <div v-text="$t('telephony.popup.selectClient')"></div>
                            <select @change="selectClient">
                                <option :value="null" v-text="$t('telephony.popup.selectClient')"></option>
                                <option v-for="client in clientsList" :key="client.id" :value="client.id" v-text="client.name"></option>
                            </select>
                        </div>
                    </div>
                </div>

                <div class="popup__container-disabled" :class="{'popup__container-disabled_loading': loading}"></div>
            </div>
        </div>
    </div>
</template>

<script>
import { mapState, mapActions, mapGetters, mapMutations } from 'vuex'

export default {
    name: 'TelephonyWindow',
    data(){
      return {
          open: false,
          loading: false,
          assetsURL: '/vue',
          filters: {
              type: null,
              status: null
          },
          filtersList: {
              type: ['missed', 'outgoing', 'incoming'],
              status: ['worked', 'not_worked']
          },
          workWithObject: null,
          opendActionCall: null,
          opendVoiceRecord: null,
          addToClientWindow: null,
      }
    },
    watch: {
        'isWindowOpen'(n) {
            if(n){
                this.workWithObject = n.item;
                this.open = true;
                if(n.addToClientListOpen) this.addToClient();
                if(n.voiceRecordOpen) this.opendVoiceRecord = n.voiceRecordOpen;
                if(n.removeDialog) this.removeDialog();
            }
            else {
                this.open = false;
                this.opendActionCall = null;
                this.opendVoiceRecord = null;
                this.addToClientWindow = null;
            }
        }
    },
    computed: {
        ...mapGetters({
            isWindowOpen: 'popup/isTelephonyWindowOpen',
            getClientByID: 'clients/getClientByID',
            clientsList: 'clients/clientsList',
        }),
        clientName(){
            const vm = this,
                re = /(?:([\d]{1,}?))??(?:([\d]{1,3}?))??(?:([\d]{1,3}?))??(?:([\d]{2}))??([\d]{2})$/;
            return (vm.workWithObject.belongTo === 'client')? vm.workWithObject.client.name : vm.workWithObject.phone.replace(re, (all, a, b, c, d, e) => ( a ? a + " " : "" ) + ( b ? b + " " : "" ) + ( c ? c + "-" : "" ) + ( d ? d + "-" : "" ) + e);
        },
        clientExist(){
            return this.workWithObject.belongTo === 'client';
        },
        history(){
            const vm = this;

            return [...this.workWithObject.history].reverse().filter(item => {
                if(vm.filters.type && item.type !== vm.filters.type) return;
                else if(vm.filters.status && item.status !== vm.filters.status) return;
                return true;
            });
        },
        employee(){
            const vm = this;
            return (vm.workWithObject.belongTo === 'client')? vm.workWithObject.client.employee.name || '' : '';
        }
    },
    created() {
        this.assetsURL = (process.env.MODE !== 'cordova' && process.env.MODE !== 'electron')? '/vue' : './vue';
    },
    methods: {
        ...mapMutations({
            openWindow: 'popup/open',
            closeWindow: 'popup/close',
            changeWindowProps: 'popup/changeWindowProps',
            addContactPerson: 'clients/addContactPerson',
        }),
        ...mapActions({
            addNewClient: 'clients/addNewClient',
            addItemToClient: 'clients/addTelephonyItemToClient',
            removeDialogAction: 'clients/removeTelephonyDialog',
            changeCallStatus: 'clients/changeCallStatus',
        }),
        close(){
            this.closeWindow('telephony');
        },
        itemDate(date){
            return `${date.getDate()}.${date.getMonth() + 1}`;
        },
        openCallActions(call){
            this.opendActionCall = call;
        },
        closeCallActions(e){
            if(e.target === this.$refs.telephonyCallActions){
                this.opendActionCall = null;
            }
        },
        closeAddToClient(e){
            if(e === null || e.target === this.$refs.telephonyAddToClient){
                this.addToClientWindow = null;
            }
        },
        closeVoiceRecord(e){
            if(e === 'clean' || e.target === this.$refs.telephonyVoiceRecord){
                this.changeWindowProps({name: 'telephony', props: {item: this.workWithObject}});
                this.opendVoiceRecord = null;
            }
        },
        openVoiceRecord(){
            if(this.opendActionCall.record){
                this.opendVoiceRecord = this.opendActionCall;
                this.opendActionCall = null;
            }
        },
        openClient(){
            if(this.workWithObject.belongTo === 'client' && this.workWithObject.client.id){
                let clientID = this.workWithObject.client.id;
                this.close();
                let section = null,
                    settings = localStorage.getItem('ClientSettings');
                if(settings){
                    try{
                        settings = JSON.parse(settings);
                    }catch (e) {
                        console.log(e);
                    }
                    if(settings.section) section = settings.section;
                }
                this.openWindow({name: 'client', props: {clientID, mode: 'view', section}});
            }
        },
        openCreateClient(){
            if(this.workWithObject.belongTo !== 'client'){
                this.addNewClient()
                    .then(clientID => {
                        this.addContactPerson({clientID, name:'', position:'', phone: this.workWithObject.phone});
                        this.openWindow({name: 'client', props: {clientID, mode: 'edit', section: 'data', createClientFromTelephony: this.workWithObject}})
                    });
            }
        },
        addToClient(){
            this.addToClientWindow = true;
            this.changeWindowProps({name: 'telephony', props: {item: this.workWithObject}});
        },
        selectClient(e){
            if(e && e.target && e.target.value){
                this.addToClientWindow = null;
                this.loading = true;
                this.addItemToClient({phone: this.workWithObject.phone, toClientID: e.target.value})
                    .then(() => this.loading = false)
                    .catch((error) => {
                        if(error.status === 403) this.openWindow({name: 'denied'});
                        this.loading = false;
                    });
                this.close();
            }
        },
        removeDialog(){
            this.changeWindowProps({name: 'telephony', props: {item: this.workWithObject}});
            this.openWindow({name: 'confirm', props: {message: this.$t('telephony.popup.removeDialogConfirm'), confirm: () => {
                this.loading = true;
                this.removeDialogAction(this.workWithObject)
                    .then(() => {
                        this.loading = false;
                        this.close();
                    })
                    .catch((error) => {
                        if(error.status === 403) this.openWindow({name: 'denied'});
                        this.loading = false;
                    });
            }}});
        },
        changeStatus(){
            let call = this.opendActionCall;
            if(call){
                this.loading = true;
                this.changeCallStatus({callID: call.id, status: (call.status === 'worked')? 'not_worked':'worked'})
                    .then(() => {
                        this.loading = false;
                        this.opendActionCall = null;
                    })
                    .catch((error) => {
                        if(error.status === 403) this.openWindow({name: 'denied'});
                        this.loading = false;
                    });
            }
        },
        callDate(call){
            return `${('0'+call.created.getHours()).slice(-2)}:${('0'+call.created.getMinutes()).slice(-2)}`;
        }
    }
}
</script>