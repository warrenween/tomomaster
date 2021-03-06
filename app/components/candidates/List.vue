<template>
    <div>
        <div
            class="container section section--status">
            <div class="row">
                <div class="col-12">
                    <h3 class="section-title">
                        <i class="tm-bolt color-pink" />
                        <span>Network Status</span>
                    </h3>
                </div>
                <div class="col-md-6 col-lg-3">
                    <b-card class="tomo-card">
                        <h6 class="tomo-card__title">Current Block</h6>
                        <p class="tomo-card__text">
                            #{{ chainConfig.blockNumber }}
                        </p>
                    </b-card>
                </div>
                <div class="col-md-6 col-lg-3">
                    <b-card class="tomo-card tomo-card">
                        <h6 class="tomo-card__title">Block Time</h6>
                        <p class="tomo-card__text">{{ chainConfig.blockTime }}.00 s</p>
                    </b-card>
                </div>
                <div class="col-md-6 col-lg-3">
                    <b-card class="tomo-card tomo-card">
                        <h6 class="tomo-card__title">Epoch</h6>
                        <p class="tomo-card__text">
                            #{{ Math.floor(chainConfig.blockNumber / chainConfig.epoch) + 1 }}</p>
                    </b-card>
                </div>
                <div class="col-md-6 col-lg-3">
                    <b-card class="tomo-card tomo-card">
                        <h6 class="tomo-card__title">Next Checkpoint</h6>
                        <p class="tomo-card__text">
                            <!-- eslint-disable-next-line max-len -->
                            #{{ parseInt(chainConfig.epoch) * (Math.floor(parseInt(chainConfig.blockNumber) / parseInt(chainConfig.epoch) + 1)) }}</p>
                    </b-card>
                </div>
            </div>
        </div>

        <div
            v-if="candidates.length <= 0"
            class="tomo-loading"/>

        <div
            v-else
            class="container">
            <div class="row">
                <div class="col-12">
                    <h3 class="section-title">
                        <i class="tm-flag color-yellow" />
                        <span>Candidates ({{ totalRows }})</span>
                    </h3>
                </div>
            </div>
            <b-table
                :items="sortedCandidates"
                :fields="fields"
                :current-page="currentPage"
                :per-page="perPage"
                :sort-by.sync="sortBy"
                :sort-desc.sync="sortDesc"
                :class="'tomo-table tomo-table--candidates ' + tableCssClass"
                empty-text="There are no candidates to show"
                stacked="md" >

                <template
                    slot="address"
                    slot-scope="data">
                    <router-link
                        :to="'/candidate/' + data.item.address"
                        class="text-truncate">
                        {{ data.item.address }}
                    </router-link>
                </template>

                <template
                    slot="cap"
                    slot-scope="data">{{ formatCurrencySymbol(formatBigNumber(data.item.cap, 2)) }}
                </template>

                <template
                    slot="latestSignedBlock"
                    slot-scope="data">#{{ data.item.latestSignedBlock || 0 }}
                </template>

                <template
                    slot="status"
                    slot-scope="data">
                    <div class="mt-2 mt-lg-0">
                        <span
                            :class="'tomo-chip '
                                + (data.item.status === 'PROPOSED' || data.item.status === 'MASTERNODE' ?
                            'tomo-chip--primary' : 'tomo-chip--accent') ">
                            {{ data.item.status.toUpperCase() }}
                        </span>
                    </div>
                </template>

                <template
                    slot="action"
                    slot-scope="data">
                    <b-button
                        v-if="data.item.status === 'PROPOSED' || data.item.status === 'MASTERNODE'"
                        variant="primary"
                        class="mt-3 mt-lg-0 vote-btn"
                        @click="onRowClick(data.item.address)">Vote</b-button>
                </template>
            </b-table>

            <b-pagination
                v-if="totalRows > 0 && totalRows > perPage"
                :total-rows="totalRows"
                :per-page="perPage"
                v-model="currentPage"
                align="center"
                class="tomo-pagination" />
        </div>
    </div>
</template>

<script>
import axios from 'axios'
import BigNumber from 'bignumber.js'
import store from 'store'

export default {
    name: 'App',
    data () {
        return {
            chainConfig: {},
            fields: [
                {
                    key: 'address',
                    label: 'Address',
                    sortable: false
                },
                {
                    key: 'name',
                    label: 'Name',
                    sortable: false
                },
                {
                    key: 'cap',
                    label: 'Capacity',
                    sortable: true
                },
                {
                    key: 'latestSignedBlock',
                    label: 'Latest Signed Block',
                    sortable: true
                },
                {
                    key: 'status',
                    label: 'Status',
                    sortable: false
                },
                {
                    key: 'action',
                    label: '',
                    sortable: false
                }
            ],
            sortBy: 'cap',
            sortDesc: true,
            isReady: false,
            account: '',
            voteActive: false,
            voteValue: 1,
            voteItem: {},
            candidates: [],
            currentPage: 1,
            perPage: 10,
            totalRows: 0,
            tableCssClass: '',
            loading: false,
            hasProposed: false,
            hasResigned: false,
            isTomonet: false
        }
    },
    computed: {
        sortedCandidates: function () {
            return this.candidates.slice().sort(function (a, b) {
                return b.cap - a.cap
            })
        }
    },
    watch: {},
    updated () {},
    created: async function () {
        let self = this
        let config = await self.appConfig()
        let account
        self.chainConfig = config.blockchain
        self.isReady = !!self.web3

        try {
            if (self.isReady) {
                let contract = await self.TomoValidator.deployed()
                if (store.get('address')) {
                    account = store.get('address').toLowerCase()
                } else {
                    account = this.$store.state.walletLoggedIn
                        ? this.$store.state.walletLoggedIn : await self.getAccount()
                }
                if (account && contract) {
                    self.isTomonet = true
                }
            }
        } catch (error) {
            console.log(error)
        }

        try {
            self.loading = true

            let candidates = await axios.get('/api/candidates')
            candidates.data.map(async (candidate, index) => {
                self.candidates.push({
                    address: candidate.candidate,
                    owner: candidate.owner.toLowerCase(),
                    status: candidate.status,
                    isMasternode: candidate.isMasternode,
                    isPenalty: candidate.isPenalty,
                    name: candidate.name || 'Anonymous',
                    cap: new BigNumber(candidate.capacity).div(10 ** 18).toNumber(),
                    latestSignedBlock: candidate.latestSignedBlock
                })
            })

            self.totalRows = self.candidates.length

            self.loading = false
            self.getTableCssClass()
        } catch (e) {
            self.loading = false
            console.log(e)
        }
    },
    mounted () { },
    methods: {
        watch: async function () {
            let contract = await self.TomoValidator.deployed()
            const allEvents = contract.allEvents({
                fromBlock: self.blockNumber,
                toBlock: 'latest'
            })

            allEvents.watch((err, res) => {
                if (err || !(res || {}).args) {
                    console.error(err, res)
                } else {
                    console.log(res)
                }
            })
        },
        getTableCssClass: function () {
            let cssClass = ''

            if (!this.candidates.length) {
                cssClass += ' tomo-table--candidates-empty'
            }

            cssClass += this.loading ? ' tomo-table--loading' : ''

            this.tableCssClass = cssClass
        },
        onRowClick (address) {
            if (this.isTomonet) {
                this.$router.push({ path: `/voting/${address}` })
            } else {
                const toastMessage = 'You can not vote at the moment. Please log in first.'
                this.$toasted.show(toastMessage, {
                    type: 'info',
                    delay: '5000'
                })
            }
        }
    }
}
</script>
