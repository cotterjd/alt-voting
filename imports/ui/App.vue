<template>
  <div>
    <button @click="startVote()">Start vote</button>
    <div style="display: flex">
      <hello v-for="key in Object.keys(votes)" :voter="key" :picks="votes[key]" />
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
  }),
  mounted () {
    this.votes = {
      joe: [`Libertarian`, `Democrat`, `Republican`, `Green Party`],
      paul: [`Democrat`, `Libertarian`, `Republican`, `Green Party`],
      dan: [`Republican`, `Libertarian`, `Democrat`, `Green Party`],
      steve: [`Libertarian`, `Republican`, `Green Party`, `Democrat`],
      pauline: [`Democrat`, `Green Party`, `Republican`, `Liberatrian`],
      jessica: [`Green Party`, `Demogract`, `Libertarian`, `Republican`],
      bob: [`Green Party`, `Republican`, `Democrat`, `Liberatarian`],
    }
  },
  computed: {},
  methods: {
    calcVotes (votesObj) {
      const topVotes = getTopVotes(votesObj)
      const [winner, losers] = getLowestAndHighest(topVotes)
      if (winner) this.winner = winner
      return Object.keys(votesObj).reduce((acc, voter) => {
        const voterPicks = votesObj[voter]
        if (losers.includes(voterPicks[0])) return { ...acc, [voter]: voterPicks.slice(1) }
        return { ...acc, [voter]: voterPicks }
      }, {})
    },
    startVote () {
      setTimeout(() => {
        if (!this.winner) {
          this.votes = this.calcVotes(this.votes)
          this.startVote()
        }
      }, 1000)
    }
  }
}
// {} -> [``]
function getTopVotes (votes) {
  return Object.keys(votes).map(key => votes[key][0])
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
