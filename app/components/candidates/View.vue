<template>
    <div
        v-if="loading"
        class="tomo-loading"/>
    <div v-else>
        <div class="container section section--candidate">
            <div class="row">
                <div class="col-12">
                    <div class="section-title">
                        <i class="tm-flag color-yellow" />
                        <span>{{ candidate.name }}</span>
                        <span class="text-truncate section-title__description">{{ candidate.address }}</span>
                        <ul class="list-inline social-links">
                            <li
                                v-for="(value, key) in candidate.socials"
                                :key="key"
                                class="list-inline-item social-links__item">
                                <a
                                    :href="value"
                                    class="social-links__link">
                                    <i :class="'social-links__icon tm-' + key" />
                                </a>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
            <b-card class="tomo-card tomo-card--animated tomo-card--candidate">
                <div class="row m-md-0">
                    <div
                        :class="'col-12 col-sm-6 col-md-4 col-lg-3 col-xl-2 tomo-info tomo-info-status--'
                        + candidate.status">
                        <p class="tomo-info__title">
                            <i class="tm-dot tomo-info__icon" />
                            <span class="tomo-info__text">Owner</span>
                        </p>
                        <p class="tomo-info__description">
                            <router-link
                                :to="'/voter/' + candidate.owner"
                                class="text-truncate">
                                {{ (candidate.owner || '').substring(0, 8) }}...
                            </router-link>
                        </p>
                    </div>
                    <div class="col-12 col-sm-6 col-md-4 col-lg-3 col-xl-2 tomo-info">
                        <p class="tomo-info__title">
                            <i class="tm-dot tomo-info__icon" />
                            <span class="tomo-info__text">Latest Signed Block</span>
                        </p>
                        <p class="tomo-info__description">
                            {{ formatNumber(candidate.latestSignedBlock) }}
                        </p>
                    </div>
                    <div class="col-12 col-sm-6 col-md-4 col-lg-3 col-xl-2 tomo-info">
                        <p class="tomo-info__title">
                            <i class="tm-dot tomo-info__icon" />
                            <span class="tomo-info__text">Capacity</span>
                        </p>
                        <p
                            id="tomo-info__description--cap"
                            class="tomo-info__description">
                            {{ formatCurrencySymbol(formatBigNumber(candidate.cap, 3)) }}
                            <b-tooltip
                                v-if="checkLongNumber(candidate.cap)"
                                ref="tooltip"
                                target="tomo-info__description--cap">
                                {{ formatCurrencySymbol(formatBigNumber(candidate.cap, 6)) }}
                            </b-tooltip>
                        </p>
                    </div>
                    <div
                        v-if="isReady"
                        class="col-12 col-sm-6 col-md-4 col-lg-3 col-xl-2 tomo-info">
                        <p class="tomo-info__title">
                            <i class="tm-arrow-up tomo-info__icon" />
                            <span class="tomo-info__text">You voted</span>
                        </p>
                        <p
                            id="tomo-info__description--you-voted"
                            class="tomo-info__description">
                            {{ formatCurrencySymbol(formatNumber(candidate.voted)) }}
                            <b-tooltip
                                v-if="checkLongNumber(candidate.voted)"
                                ref="tooltip"
                                target="tomo-info__description--you-voted">
                                {{ formatCurrencySymbol(formatBigNumber(candidate.voted, 6)) }}
                            </b-tooltip>
                        </p>
                    </div>
                    <div
                        class="col-12 col-sm-6 col-md-4 col-lg-3 col-xl-2 tomo-info">
                        <p class="tomo-info__title">
                            <i class="tm-dot tomo-info__icon" />
                            <span class="tomo-info__text">Recent Reward</span>
                        </p>
                        <p
                            id="tomo-info__description--you-rewarded"
                            class="tomo-info__description">
                            {{ formatCurrencySymbol(formatNumber(recentReward)) }}
                            <b-tooltip
                                v-if="checkLongNumber(candidate.rewarded)"
                                ref="tooltip"
                                target="tomo-info__description--you-rewarded">
                                {{ formatCurrencySymbol(formatBigNumber(candidate.rewarded, 6)) }}
                            </b-tooltip>
                        </p>
                    </div>
                    <div class="col-12 col-sm-6 col-md-4 col-lg-3 col-xl-2 m-xl-0 tomo-info">
                        <p class="tomo-info__title">
                            <i class="tm-dot tomo-info__icon" />
                            <span class="tomo-info__text">Monitor</span>
                        </p>
                        <p class="tomo-info__description">
                            {{ candidate.monitor }}
                        </p>
                    </div>
                    <div
                        v-if="isReady"
                        class="col-12 col-sm-6 col-md-4 col-lg-3 col-xl-2 m-xl-0 tomo-info">
                        <p class="tomo-info__title">
                            <i class="tm-dot tomo-info__icon" />
                            <span class="tomo-info__text">Balance</span>
                        </p>
                        <p
                            id="tomo-info__description--balance"
                            class="tomo-info__description">
                            {{ formatCurrencySymbol(formatBigNumber(candidate.balance, 3)) }}
                            <b-tooltip
                                v-if="checkLongNumber(candidate.balance)"
                                ref="tooltip"
                                target="tomo-info__description--balance">
                                {{ formatCurrencySymbol(formatBigNumber(candidate.balance, 6)) }}
                            </b-tooltip>
                        </p>
                    </div>
                    <div class="col-12 col-md-6 col-lg-6 col-xl-4 order-md-1 order-lg-0 m-xl-0 tomo-info">
                        <p class="tomo-info__title">
                            <i class="tm-dot tomo-info__icon" />
                            <span class="tomo-info__text">Hardware</span>
                        </p>
                        <p class="tomo-info__description">
                            {{ candidate.hardwareInfo }}
                        </p>
                    </div>
                    <div
                        v-for="(value, key) in candidate.dataCenterInfo"
                        :key="key"
                        class="col-12 col-sm-6 col-md-4 col-lg-3 col-xl-2 m-xl-0 tomo-info">
                        <p class="tomo-info__title">
                            <i class="tm-dot tomo-info__icon" />
                            <span class="tomo-info__text">{{ key }}</span>
                        </p>
                        <p class="tomo-info__description">
                            {{ value }}
                        </p>
                    </div>
                </div>
            </b-card>
            <div
                class="buttons text-right">
                <b-button
                    v-if="candidate.owner === account && candidate.status !== 'RESIGNED'"
                    :to="`/resign/${candidate.address}`"
                    variant="secondary">Resign</b-button>
                <b-button
                    v-if="candidate.voted > 0"
                    :to="`/unvoting/${candidate.address}`"
                    variant="secondary">Unvote</b-button>
                <b-button
                    v-if="candidate.status !== 'RESIGNED' && isTomonet"
                    :to="`/voting/${candidate.address}`"
                    variant="primary">Vote</b-button>
            </div>
        </div>
        <div
            v-if="candidate.status !== 'RESIGNED' && candidate.nodeId"
            class="container section section--hardware">
            <div class="row">
                <div class="col-12 col-lg-6">
                    <h3 class="section-title">
                        <i class="tm-cpu color-pink" />
                        <span>CPUs</span>
                    </h3>
                    <chart
                        :host="candidate.nodeId"
                        data-type="cpu"
                        class="mb-5" />
                </div>
                <div class="col-12 col-lg-6">
                    <h3 class="section-title">
                        <i class="tm-memory color-orange" />
                        <span>Memory</span>
                    </h3>
                    <chart
                        :host="candidate.nodeId"
                        data-type="memory" />
                </div>
            </div>
        </div>
        <div class="container section section--mnrewards">
            <div class="row">
                <div class="col-12">
                    <h3 class="section-title">
                        <i class="tm-gift color-purple" />
                        <span>Masternode Rewards</span>
                        <span class="text-truncate section-title__description">
                            Estimated Reward for Masternode</span>
                    </h3>
                </div>
            </div>
            <b-table
                :items="mnRewards"
                :fields="mnRewardsFields"
                :current-page="mnRewardsCurrentPage"
                :sort-by.sync="mnRewardsSortBy"
                :sort-desc.sync="mnRewardsSortDesc"
                :per-page="mnRewardsPerPage"
                :show-empty="true"
                :class="`tomo-table tomo-table--mnrewards${loading ? ' loading' : ''}`"
                empty-text="There are no rewards to show"
                stacked="md" >

                <template
                    slot="checkpoint"
                    slot-scope="data">{{ data.item.checkpoint }}
                </template>

                <template
                    slot="reward"
                    slot-scope="data">
                    {{ formatCurrencySymbol(formatNumber(data.item.reward)) }}
                </template>

                <template
                    slot="createdAt"
                    slot-scope="data">
                    <span :id="`timestamp__${data.index}`">{{ data.item.createdAt }}</span>
                    <b-tooltip :target="`timestamp__${data.index}`">
                        {{ data.item.dateTooltip }}
                    </b-tooltip>
                </template>

            </b-table>

            <b-pagination
                v-if="mnRewardsTotalRows > 0 && mnRewardsTotalRows > mnRewardsPerPage"
                :total-rows="mnRewardsTotalRows"
                :per-page="mnRewardsPerPage"
                v-model="mnRewardsCurrentPage"
                align="center"
                class="tomo-pagination" />
        </div>
        <div class="container section section-voters">
            <div class="row">
                <div class="col-12">
                    <h3 class="section-title">
                        <i class="tm-arrow-up color-pink" />
                        <span>Voters</span>
                        <span class="text-truncate section-title__description">
                            People who voted for this candidate</span>
                    </h3>
                </div>
            </div>
            <b-table
                :items="sortedVoters"
                :fields="voterFields"
                :current-page="voterCurrentPage"
                :per-page="voterPerPage"
                :show-empty="true"
                :class="`tomo-table tomo-table--voted${loading ? ' loading' : ''}`"
                empty-text="There are no voters to show"
                stacked="md" >

                <template
                    slot="id"
                    slot-scope="data">{{ data.item.id }}
                </template>

                <template
                    slot="address"
                    slot-scope="data">
                    <router-link
                        :to="'/voter/' + data.item.address"
                        class="text-truncate">
                        {{ data.item.address }}
                    </router-link>
                </template>

                <template
                    slot="cap"
                    slot-scope="data">{{ formatCurrencySymbol(formatNumber(data.item.cap)) }}
                </template>
            </b-table>

            <b-pagination
                v-if="voterTotalRows > 0 && voterTotalRows > voterPerPage"
                :total-rows="voterTotalRows"
                :per-page="voterPerPage"
                v-model="voterCurrentPage"
                align="center"
                class="tomo-pagination" />
        </div>
        <div class="container section section--txs">
            <div class="row">
                <div class="col-12">
                    <h3 class="section-title">
                        <i class="tm-time color-purple" />
                        <span>Transactions</span>
                        <span class="text-truncate section-title__description">
                            All transactions of this candidate</span>
                    </h3>
                </div>
            </div>
            <b-table
                :items="transactions"
                :fields="txFields"
                :current-page="txCurrentPage"
                :per-page="txPerPage"
                :show-empty="true"
                :class="`tomo-table tomo-table--transactions${loading ? ' loading' : ''}`"
                empty-text="There are no transactions to show"
                stacked="md" >

                <template
                    slot="id"
                    slot-scope="data">{{ data.item.id }}
                </template>

                <template
                    slot="voter"
                    slot-scope="data">
                    <router-link
                        :to="'/voter/' + data.item.voter"
                        class="text-truncate">
                        {{ data.item.voter }}
                    </router-link>
                </template>

                <template
                    slot="event"
                    slot-scope="data">
                    <span :class="'fw-600 ' + getEventClass(data.item.event)">{{ data.item.event }}</span>
                </template>

                <template
                    slot="cap"
                    slot-scope="data">
                    {{ isNaN(data.item.cap) ? '---' : formatCurrencySymbol(data.item.cap) }}
                </template>

                <template
                    slot="createdAt"
                    slot-scope="data">
                    <span :id="`timestamp__${data.index}`">{{ data.item.createdAt }}</span>
                    <b-tooltip :target="`timestamp__${data.index}`">
                        {{ data.item.dateTooltip }}
                    </b-tooltip>
                </template>

                <template
                    slot="action"
                    slot-scope="data">
                    <a
                        v-b-tooltip.hover.right
                        :href="`${config.explorerUrl}/txs/${data.item.tx}`"
                        title="View on TomoScan"
                        target="_blank">
                        <i class="tm-eye" />
                        <span>View on TomoScan</span>
                    </a>
                </template>
            </b-table>

            <b-pagination
                v-if="txTotalRows > 0 && txTotalRows > txPerPage"
                :total-rows="txTotalRows"
                :per-page="txPerPage"
                v-model="txCurrentPage"
                align="center"
                class="tomo-pagination" />
        </div>
    </div>
</template>
<script>
import axios from 'axios'
import BigNumber from 'bignumber.js'
import Chart from '../Chart.vue'
import moment from 'moment'
import store from 'store'

export default {
    name: 'App',
    components: {
        chart: Chart
    },
    data () {
        return {
            isReady: false,
            account: '',
            voteActive: false,
            voteValue: 1,
            unvoteValue: 1,
            recentReward: 0,
            config: {},
            voters: [],
            transactions: [],
            mnRewards: [],
            candidate: {
                address: this.$route.params.address.toLowerCase(),
                name: '',
                balance: '',
                status: 'active',
                cap: 0,
                latestBlock: '',
                latestSignedBlock: 0,
                rewarded: 0,
                hardwareInfo: '',
                dataCenterInfo: {},
                socials: {},
                voted: 0
            },
            mnRewardsFields: [
                {
                    key: 'epoch',
                    label: 'Epoch',
                    sortable: false
                },
                {
                    key: 'signNumber',
                    label: 'Sign Number',
                    sortable: false
                },
                {
                    key: 'reward',
                    label: 'Reward',
                    sortable: false
                },
                {
                    key: 'createdAt',
                    label: 'Age',
                    sortable: false
                }
            ],
            mnRewardsCurrentPage: 1,
            mnRewardsSortBy: 'epoch',
            mnRewardsPerPage: 10,
            mnRewardsSortDesc: true,
            mnRewardsTotalRows: 0,
            voterFields: [
                {
                    key: 'address',
                    label: 'Address',
                    sortable: true
                },
                {
                    key: 'cap',
                    label: 'Capacity',
                    sortable: true
                }
            ],
            voterSortBy: 'cap',
            voterSortDesc: true,
            voterCurrentPage: 1,
            voterPerPage: 10,
            voterTotalRows: 0,
            txFields: [
                {
                    key: 'voter',
                    label: 'Voter',
                    sortable: true
                },
                {
                    key: 'event',
                    label: 'Event',
                    sortable: true
                },
                {
                    key: 'cap',
                    label: 'Capacity',
                    sortable: true
                },
                {
                    key: 'createdAt',
                    label: 'Age',
                    sortable: false
                },
                {
                    key: 'action',
                    label: '',
                    sortable: false
                }
            ],
            txSortBy: 'cap',
            txSortDesc: true,
            txCurrentPage: 1,
            txPerPage: 10,
            txTotalRows: 0,
            loading: false,
            chartLoading: false,
            cpu0Series: [],
            isTomonet: false
        }
    },
    computed: {
        sortedVoters: function () {
            return this.voters.slice().sort(function (a, b) {
                return b.cap - a.cap
            })
        }
    },
    watch: {
        $route (to, from) {
            this.candidate.address = to.params.address.toLowerCase()
            this.getCandidateData()
        }
    },
    updated () {},
    created: async function () {
        let self = this
        self.config = await self.appConfig()
        self.isReady = !!self.web3

        self.getCandidateData()
    },
    mounted () {},
    methods: {
        getEventClass (event) {
            let clazz = ''
            if (event === 'Unvote' || event === 'Resign') {
                clazz = 'color-pink'
            }

            return clazz
        },
        getDate (date) {
            return date
        },
        async getCandidateData () {
            let self = this
            try {
                if (self.isReady) {
                    let contract = await self.TomoValidator.deployed()
                    if (store.get('address')) {
                        self.account = store.get('address').toLowerCase()
                    } else {
                        self.account = this.$store.state.walletLoggedIn
                            ? this.$store.state.walletLoggedIn : await self.getAccount()
                    }
                    if (self.account && contract) {
                        self.isTomonet = true
                    }
                }
            } catch (error) {
                console.log(error)
            }

            try {
                let address = self.candidate.address

                self.loading = true
                // Get all information at the same time
                const promises = await Promise.all([
                    await axios.get(`/api/candidates/${address}`),
                    await axios.get(`/api/candidates/${address}/voters`),
                    await axios.get(`/api/transactions/candidate/${address}`)
                ])

                // Get candidate's information
                let c = promises[0]

                if (c.data) {
                    let data = c.data
                    self.candidate.name = data.name ? data.name : 'Anonymous Candidate'
                    self.candidate.status = data.status
                    self.candidate.nodeId = data.nodeId
                    self.candidate.monitor = (data.nodeId) ? 'ON' : 'OFF'
                    self.candidate.owner = data.owner
                    self.candidate.cap = new BigNumber(data.capacity).div(10 ** 18).toNumber()
                    self.candidate.rewarded = 0
                    self.candidate.latestBlock = '0'
                    self.candidate.latestSignedBlock = data.latestSignedBlock
                    self.candidate.hardwareInfo = data.hardware || 'N/A'
                    self.candidate.dataCenterInfo = {
                        name: (data.dataCenter || {}).name || 'N/A',
                        location: (data.dataCenter || {}).location || 'N/A'
                    }
                    self.candidate.socials = data.socials
                }

                if (self.web3) {
                    self.web3.eth.getBalance(self.candidate.address, function (a, b) {
                        self.candidate.balance = new BigNumber(b).div(10 ** 18)
                        if (a) {
                            console.log('got an error', a)
                        }
                    })
                }

                // Voter table
                let voters = promises[1]

                let youVoted = new BigNumber(0)
                voters.data.map((v, idx) => {
                    self.voters.push({
                        address: v.voter,
                        cap: new BigNumber(v.capacity).div(10 ** 18).toNumber()
                    })

                    if (v.voter === self.account) {
                        youVoted = youVoted.plus(v.capacity)
                    }
                })

                if (self.account && self.web3) {
                    try {
                        let contract = await self.TomoValidator.deployed()
                        youVoted = await contract.getVoterCap(address, self.account)
                        self.candidate.cap = await contract.getCandidateCap(address).div(1e18).toNumber()
                    } catch (e) {}
                }

                self.candidate.voted = youVoted.div(10 ** 18).toNumber()

                self.voterTotalRows = self.voters.length

                // Get transaction table
                let txs = promises[2]

                txs.data.map((tx, idx) => {
                    self.transactions.push({
                        tx: tx.tx,
                        voter: tx.voter,
                        candidate: tx.candidate,
                        event: tx.event,
                        cap: new BigNumber(tx.capacity).div(10 ** 18).toNumber(),
                        createdAt: moment(tx.createdAt).fromNow(),
                        dateTooltip: moment(tx.createdAt).format('lll')
                    })
                })

                self.txTotalRows = self.transactions.length

                // Masternode reward table
                let mnRewards = await axios.get(`/api/candidates/${address}/${self.candidate.owner}/getRewards`)

                mnRewards.data.map((r) => {
                    self.mnRewards.push({
                        epoch: r.epoch,
                        signNumber: r.signNumber,
                        reward: new BigNumber(r.reward).toFixed(6),
                        createdAt: moment(r.createdAt).fromNow(),
                        dateTooltip: moment(r.createdAt).format('lll')
                    })
                })

                self.recentReward = (self.mnRewards[0] || {}).reward || 0

                self.mnRewardsTotalRows = self.mnRewards.length

                self.loading = false
            } catch (e) {
                self.loading = false
                console.log(e)
            }
        }
    }
}
</script>
