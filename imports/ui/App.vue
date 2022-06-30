<template>
  <div>
    <button @click="startVote()">Simulate Next Round</button>
    <div style="display: flex">
      <hello v-for="key in Object.keys(votes)" :voter="key" :picks="votes[key]" />
      <div>
        <div v-for="key in Object.keys(countObj)">{{key}}: {{countObj[key]}}</div>
        <div v-for="loser in losers"><s>{{loser}}</s></div>
      </div>
    </div>
  </div>
</template>

<script>
import Hello from './components/Hello.vue'
import Info from './components/Info.vue'
import series from 'promise.series'
import * as R from 'ramda' // TODO: import these individually

export default {
  components: {
    Hello,
    Info,
  },
  data: () => ({
    winner: ``,
    votes: {},
    countObj: {},
    losers: [],
    VoteString: ``,
  }),
  mounted () {
    const initialVotes = {
      joe: [`Republican`, `Democrat`, `Libertarian`, `Green Party`],
      paul: [`Green Party`, `Libertarian`, `Republican`, `Democrat`],
      dan: [`Republican`, `Libertarian`, `Democrat`, `Green Party`],
      steve: [`Libertarian`, `Republican`, `Green Party`, `Democrat`],
      pauline: [`Democrat`, `Green Party`, `Republican`, `Libertarian`],
      jessica: [`Green Party`, `Democrat`, `Libertarian`, `Republican`],
      bob: [`Green Party`, `Republican`, `Democrat`, `Libertarian`],
    }
    this.votes = initialVotes
    this.countObj = getCountObj(initialVotes) 
  },
  computed: {},
  methods: {
    // TODO: fix not stopping after majority
    calcVotes (votesObj) {
      const topVotes = getTopVotes(votesObj)
      const [winner, losers] = getLowestAndHighest(topVotes)
      const newLosers = R.uniq([...this.losers, ...losers])
      console.log(newLosers)
      if (winner) this.winner = winner
      this.losers = newLosers
      return Object.keys(votesObj).reduce((acc, voter) => ({ ...acc, [voter]: getUpdatedVoterPicks(newLosers, votesObj[voter])}), {})
    },
    startVote () {
      if (!this.winner) {
        const newVotes = this.calcVotes(this.votes)
        this.votes = newVotes
        const countObj = getCountObj(newVotes)
        this.countObj = countObj
      }
    },
    // {} -> `` 
  }
}
function getUpdatedVoterPicks (losers, voterPicks) {
  if (!losers.includes(voterPicks[0])) return voterPicks
  return getUpdatedVoterPicks(losers, voterPicks.slice(1))
}
// {} -> {} 
function getCountObj (votesObj) {
  // [``]
  const topVotes = getTopVotes(votesObj)
  // {}
  const countObj = topVotes.reduce((acc, str) => {
    if (acc[str]) return { ...acc, [str]: acc[str]+1 }
    return { ...acc, [str]: 1 }
  }, {})
  return countObj
}
// {} -> [``]
function getTopVotes (votesObj) {
  return Object.keys(votesObj).map(key => votesObj[key][0])
}
// [``] -> ``
function getWinner (groupedVotes, totalVotes) {
  const highest = groupedVotes.reduce((high, supporters) => supporters.length > high.length ? supporters : high, groupedVotes[0]) 
  const lengthNeeded = Math.floor(totalVotes / 2 + 1)
  return highest.length >= lengthNeeded ? highest[0] : ``
}
// [``] -> [``, []] 
function getLowestAndHighest (votes) {
  const sortFunc = (a, b) => a === b ? -1 : 1 
  // [[]]
  const grouped = R.groupWith(R.equals, R.sort(sortFunc, votes))
  const winner = getWinner(grouped, votes.length)
  // [``]
  const losers = R.flatten(grouped.reduce((acc, supporters) => {
    if (supporters.length === acc[0].length) return R.uniq([...acc, supporters])
    if (supporters.length < acc[0].length) return [supporters]
    return acc
  }, [grouped[0]]))
  return [winner, losers]
}
</script>

<style>
  body {
    font-family: sans-serif;
    padding: 10px;
  }
</style>
