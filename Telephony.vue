<template>
  <div class="clients__page-telephony telephony-page">
      <div class="page-table">
          <div class="table-cover">
              <div class="clients__table table">
                  <div class="table__row">

                      <div class="table__cell">
                          <div>{{ $t('telephony.client.title') }}</div>
                      </div>
                      <div class="table__cell table__cell_tab" :class="{'table__cell_active-border': filters.type}">
                          <span class="table__cell-sort">
                            {{ $t('telephony.types.title') }}
                            <select v-model="filters.type">
                                <option disabled>{{ $t('clients.table.status.first') }}</option>
                                <option v-for="item in filtersList.type" :key="item" :value="item">{{ $t('telephony.types.' + item) }}</option>
                                <option :value="null">{{ $t('clients.table.status.all') }}</option>
                            </select>
                          </span>
                      </div>
                      <div class="table__cell table__cell_client-date-time">
                          <div>{{ $t('telephony.date.title') }}</div>
                      </div>
                      <div class="table__cell table__cell_telephony-status" :class="{'table__cell_active-border': filters.status}">
                          <span class="table__cell-sort">
                            {{ $t('telephony.status.title') }}
                            <select v-model="filters.status">
                                <option disabled>{{ $t('clients.table.status.first') }}</option>
                                <option v-for="item in filtersList.status" :key="item" :value="item">{{ $t('telephony.status.' + item) }}</option>
                                <option :value="null">{{ $t('clients.table.status.all') }}</option>
                            </select>
                          </span>
                      </div>
                      <div class="table__cell table__cell_tab" :class="{'table__cell_active-border': filters.employee}">
                          <span class="table__cell-sort">
                            {{ $t('telephony.employee.title') }}
                            <select v-model="filters.employee">
                                <option disabled>{{ $t('clients.table.status.first') }}</option>
                                <option v-for="item in employeeList" :key="item.id" :value="item.id">{{ item.name }}</option>
                                <option :value="null">{{ $t('clients.table.status.all') }}</option>
                            </select>
                          </span>
                      </div>
                      <div class="table__cell table__cell_tab table__cell_client-date">
                          <div>{{ $t('telephony.calls.title') }}</div>
                      </div>
                      <div class="table__cell table__cell_tab table__cell_telephony-dur">
                          <div>{{ $t('telephony.duration.title') }}</div>
                      </div>

                  </div>

                  <TelephonyItem
                      v-for="item in telephony"
                      :key="getTelephonyItemKey(item)"
                      :item="item"
                      :filters="filters"
                      @click.native="open(item)"
                      @contextmenu.native.prevent="openContextMenu($event, item)"
                  />

              </div>
          </div>

          <div v-show="!telephony.length" class="no-content" v-text="$t('noContent.clients')"></div>

          <div class="telephony-page__total clear">
              <div>{{ $t('telephony.sum') }}: {{ telephony.length }}</div>
              <div>{{ $t('telephony.sumCalls') }}: {{ callsSum }}</div>
              <div>{{ minutesSum + ' ' + $t('telephony.minutesSum') }}</div>
          </div>
      </div>

      <div class="contextmenu" ref="contextMenu" :class="[contextmenu.open? 'contextmenu_open':'']" :style="{left: contextmenu.left+'px', top: contextmenu.top+'px', opacity: contextmenu.opacity}">
          <div class="contextmenu__item" @click="changeStatus" v-text="$t('telephony.actions.changeStatus', {status: contextMenuStatus})"></div>
          <div class="contextmenu__item" v-text="$t('telephony.popup.toCard')" v-show="contextMenuToCardShow" @click="openClient"></div>
          <div class="contextmenu__item" v-text="$t('telephony.popup.createClient')" v-show="!contextMenuToCardShow" @click="openCreateClient"></div>
          <div class="contextmenu__item" v-text="$t('telephony.popup.addToClient')" v-show="!contextMenuToCardShow" @click="addNumberToClient"></div>
          <div class="contextmenu__item contextmenu__item_na" v-text="$t('telephony.actions.call')"></div>
          <div class="contextmenu__item" v-text="$t('telephony.actions.makeTask')" v-show="contextMenuToCardShow" @click="makeTask"></div>
          <div class="contextmenu__item" v-text="$t('telephony.actions.play')" :class="{'contextmenu__item_na': !contextMenuVoicePlayShow}" @click="openVoiceRecord"></div>
          <div class="contextmenu__item" v-text="$t('telephony.popup.removeDialog')" @click="removeDialog"></div>
      </div>

      <TelephonyWindow/>

      <!-- mobile filters -->
      <div class="clients__filter-tab clear">
          <div class="clients__filter-tab-item">
                <span class="table__cell-sort">
                    {{ $t('telephony.types.title') }}
                    <select v-model="filters.type">
                        <option disabled>{{ $t('clients.table.status.first') }}</option>
                        <option v-for="item in filtersList.type" :key="item" :value="item">{{ $t('telephony.types.' + item) }}</option>
                        <option :value="null">{{ $t('clients.table.status.all') }}</option>
                    </select>
                </span>
          </div>
          <div class="clients__filter-tab-item">
                <span class="table__cell-sort">
                    {{ $t('telephony.employee.title') }}
                    <select v-model="filters.employee">
                        <option disabled>{{ $t('clients.table.status.first') }}</option>
                        <option v-for="item in employeeList" :key="item.id" :value="item.id">{{ item.name }}</option>
                        <option :value="null">{{ $t('clients.table.status.all') }}</option>
                    </select>
                </span>
          </div>
          <div class="clients__filter-tab-item">
                <span class="table__cell-sort">
                    {{ $t('telephony.status.title') }}
                    <select v-model="filters.status">
                        <option disabled>{{ $t('clients.table.status.first') }}</option>
                        <option v-for="item in filtersList.status" :key="item" :value="item">{{ $t('telephony.status.' + item) }}</option>
                        <option :value="null">{{ $t('clients.table.status.all') }}</option>
                    </select>
                </span>
          </div>
      </div>

  </div>
</template>

<script>
import { mapState, mapGetters, mapActions, mapMutations } from 'vuex'
import TelephonyItem from 'src/components/telephony/Item.vue'
import TelephonyWindow from 'src/components/telephony/Popup.vue'

export default {
    name: 'Telephony',
    props: ['sort', 'search'],
    data() {
        return {
            assetsURL: '/vue',
            filters: {
                type: null,
                status: null,
                employee: null
            },
            filtersList: {
                type: ['missed', 'outgoing', 'incoming'],
                status: ['worked', 'not_worked']
            },
            contextmenu: {
                open: false,
                left: 0,
                top: 0,
                opacity: 0,
                object: null
            }
        }
    },
    components: {
        TelephonyItem,
        TelephonyWindow
    },
    created() {
        this.assetsURL = (process.env.MODE !== 'cordova' && process.env.MODE !== 'electron')? '/vue' : './vue';
        this.loadTelephony();
        document.addEventListener('click', this.contextMenuClose);
    },
    beforeDestroy(){
        document.removeEventListener('click', this.contextMenuClose);
    },
    computed: {
        ...mapGetters({
            employeeList: 'employee/list',
            telephonyList: 'clients/telephony',
            getClientByID: 'clients/getClientByID',
        }),
        contextMenuStatus(){
            let obj = this.contextmenu.object;
            if(obj){
                let call = this.getViewCallItem(obj);
                return this.$t('telephony.status.'+((call.status === 'worked')? 'not_worked':'worked'));
            }
            else return '-';
        },
        contextMenuToCardShow(){
            let obj = this.contextmenu.object;
            return obj? obj.belongTo === 'client' : false;
        },
        contextMenuVoicePlayShow(){
            let obj = this.contextmenu.object;
            if(obj){
                let call = this.getViewCallItem(obj);
                return call.record;
            }
            else return false;
        },

        telephony(){
            const {sort, search} = this;

            let items = [...this.telephonyList.filter(item => {
                if(search.trim().length){
                    let compareWith = (item.belongTo === 'client')? item.client.name : item.phone;
                    return compareWith.toLowerCase().includes(search.trim().toLowerCase());
                }
                return this.getClientFilteredHistoryItems(item).length;
            })].sort((t1, t2) => {

                let t1Item = this.getClientFilteredHistoryItems(t1);
                t1Item = t1Item[t1Item.length - 1];

                let t2Item = this.getClientFilteredHistoryItems(t2);
                t2Item = t2Item[t2Item.length - 1];

                if(sort.by === 'status') return (this.$t(`telephony.status.${t1Item.status}`) > this.$t(`telephony.status.${t2Item.status}`))? 1 : -1;
                else if(sort.by === 'date') return t2Item.created - t1Item.created;
                else if(sort.by === 'type') return (this.$t(`telephony.types.${t1Item.type}`) > this.$t(`telephony.types.${t2Item.type}`))? 1 : -1;
                else if(sort.by === 'client') return ((t2.phone || t2.client.name) > (t1.phone || t1.client.name))? 1 : -1;
                else if(sort.by === 'employee') return ((t1.phone? '' : t1.client.employee.name) > (t2.phone? '' : t2.client.employee.name))? 1 : -1;
                else if(sort.by === 'duration') return (t1Item.duration > t2Item.duration)? 1 : -1;

                return 0;
            });

            if(sort.dir === 'DESC') items.reverse();

            return items;
        },
        minutesSum(){
            return this.telephony.reduce((sum, current) => {
                let t1Item = this.getClientFilteredHistoryItems(current);
                t1Item = t1Item[t1Item.length - 1];
                return sum + t1Item.duration;
            }, 0);
        },
        callsSum(){
            return this.telephony.reduce((sum, current) => {
                return sum + current.history.length;
            }, 0);
        }
    },
    methods: {
        ...mapMutations({
            openWindow: 'popup/open',
            addContactPerson: 'clients/addContactPerson',
        }),
        ...mapActions({
            loadTelephony: 'clients/loadTelephony',
            addNewClient: 'clients/addNewClient',
            changeCallStatus: 'clients/changeCallStatus',
        }),
        getViewCallItem(client){
            return [...this.getClientFilteredHistoryItems(client)].pop();
        },

        open(item, {addToClientListOpen = false, voiceRecordOpen = false, removeDialog = false} = {}){
            this.openWindow({name: 'telephony', props: {item, addToClientListOpen, voiceRecordOpen, removeDialog}});
        },
        getTelephonyItemKey(item){
            return (item.belongTo === 'client')? item.client.id : item.phone;
        },
        getClientFilteredHistoryItems(client){
            return client.history.filter(historyItem => {
                if(this.filters.type && historyItem.type !== this.filters.type) return;
                if(this.filters.status && historyItem.status !== this.filters.status) return;
                if(this.filters.employee) {
                    if(client.belongTo === 'client'){
                        if(client.client.employee.id !== this.filters.employee) return;
                    }
                    else return;
                }
                return true;
            });
        },

        changeStatus(){
            let obj = this.contextmenu.object;
            if(obj){
                let call = this.getViewCallItem(obj);
                if(call){
                    this.changeCallStatus({callID: call.id, status: (call.status === 'worked')? 'not_worked':'worked'})
                        .then(() => {
                            console.log('done');
                        });
                }
            }
        },
        makeTask(){
            let obj = this.contextmenu.object;
            if(obj){
                if(obj.belongTo === 'client' && obj.client.id){
                    let clientID = obj.client.id;
                    this.openWindow({name: 'client', props: {clientID, mode: 'view', section: 'tasks'}});
                }
            }
        },
        openClient(){
            let obj = this.contextmenu.object;
            if(obj){
                if(obj.belongTo === 'client' && obj.client.id){
                    let clientID = obj.client.id;
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
            }
        },
        openVoiceRecord(){
            if(this.contextMenuVoicePlayShow){
                let obj = this.contextmenu.object;
                if(obj) this.open(obj, {voiceRecordOpen: this.getViewCallItem(obj)});
            }
        },
        removeDialog(){
            let obj = this.contextmenu.object;
            if(obj) this.open(obj, {removeDialog: true});
        },
        openCreateClient(){
            let obj = this.contextmenu.object;
                if(obj){
                if(obj.belongTo !== 'client'){
                    this.addNewClient()
                        .then(clientID => {
                            this.addContactPerson({clientID, name:'', position:'', phone: obj.phone});
                            this.openWindow({name: 'client', props: {clientID, mode: 'edit', section: 'data', createClientFromTelephony: obj}})
                        });
                }
            }
        },
        addNumberToClient(){
            let obj = this.contextmenu.object;
            if(obj && obj.belongTo !== 'client') this.open(obj, {addToClientListOpen: true});
        },

        openContextMenu(e, item){
            let click = {top: e.clientY, left: e.clientX};
            let window = {width: document.documentElement.clientWidth, height: document.documentElement.clientHeight};

            this.contextmenu.object = item;
            this.contextmenu.open = true;

            setTimeout(() => {
                this.contextmenu.top = (window.height - click.top >= this.$refs.contextMenu.clientHeight)? click.top : window.height - this.$refs.contextMenu.clientHeight;
                this.contextmenu.left = ((window.width - click.left >= this.$refs.contextMenu.clientWidth)? click.left : window.width - this.$refs.contextMenu.clientWidth) - 60;
                this.contextmenu.opacity = 1;
            });
        },
        contextMenuClose(){
            setTimeout(() => {
                this.contextmenu.open = false;
                this.contextmenu.opacity = 0;
                this.contextmenu.object = null;
            });
        }
    }
}
</script>
