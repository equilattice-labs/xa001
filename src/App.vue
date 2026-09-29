<script setup>
import { computed, ref } from 'vue'

const tabs = ['All markets', 'Trending', 'Crypto', 'Culture', 'Sports']
const activeTab = ref('All markets')
const query = ref('')
const connected = ref(false)
const walletOpen = ref(false)
const selectedMarket = ref(null)
const selectedSide = ref('YES')
const stake = ref(25)
const toast = ref('')
const contractAddress = '0x0813A921Ad488C0604d63a4D673536730fF7E725'
const contractExplorerUrl = `https://explorer.testnet.chain.robinhood.com/address/${contractAddress}`

const markets = [
  { id: 1, icon: 'sun', category: 'Crypto', title: 'Will BTC break $100k before July?', subtitle: 'Bitcoin // July 01, 2025', yes: 68, volume: '$482,910', change: '+8.2%', hot: true },
  { id: 2, icon: 'cloud', category: 'Culture', title: 'Will the next big game drop this summer?', subtitle: 'Gaming // August 31, 2025', yes: 43, volume: '$139,420', change: '-2.4%', hot: false },
  { id: 3, icon: 'coin', category: 'Crypto', title: 'ETH flips BTC on daily fees?', subtitle: 'Ethereum // June 30, 2025', yes: 21, volume: '$98,260', change: '+4.7%', hot: true },
  { id: 4, icon: 'star', category: 'Sports', title: 'Will the home team take the crown?', subtitle: 'NBA Finals // June 18, 2025', yes: 57, volume: '$311,850', change: '+1.3%', hot: false },
  { id: 5, icon: 'flower', category: 'Culture', title: 'Will a pixel classic get a remake?', subtitle: 'Entertainment // September 12, 2025', yes: 76, volume: '$76,103', change: '+10.1%', hot: true },
  { id: 6, icon: 'pipe', category: 'Crypto', title: 'Robinhood Chain TVL over $1B?', subtitle: 'Robinhood Chain // December 31, 2025', yes: 34, volume: '$205,600', change: '-0.8%', hot: false },
]

const filteredMarkets = computed(() => {
  const normalized = query.value.trim().toLowerCase()
  return markets.filter((market) => {
    const matchesTab = activeTab.value === 'All markets' || (activeTab.value === 'Trending' ? market.hot : market.category === activeTab.value)
    const matchesSearch = !normalized || `${market.title} ${market.subtitle}`.toLowerCase().includes(normalized)
    return matchesTab && matchesSearch
  })
})

const payout = computed(() => {
  if (!selectedMarket.value) return 0
  const probability = selectedSide.value === 'YES' ? selectedMarket.value.yes : 100 - selectedMarket.value.yes
  return ((Number(stake.value || 0) / Math.max(probability, 1)) * 100).toFixed(2)
})

function openTrade(market, side = 'YES') {
  selectedMarket.value = market
  selectedSide.value = side
  stake.value = 25
}

function closeTrade() {
  selectedMarket.value = null
}

function connectWallet() {
  connected.value = true
  walletOpen.value = false
  toast.value = 'Wallet connected / 0x7A...BEEF'
  window.setTimeout(() => (toast.value = ''), 2600)
}

function confirmTrade() {
  if (!connected.value) {
    walletOpen.value = true
    return
  }
  toast.value = `${selectedSide.value} position queued / ${stake.value} USDC`
  closeTrade()
  window.setTimeout(() => (toast.value = ''), 3000)
}
</script>

<template>
  <div class="app-shell">
    <div class="pixel-sky" aria-hidden="true">
      <span class="cloud cloud-a"></span><span class="cloud cloud-b"></span>
      <span class="coin coin-a">$</span><span class="coin coin-b">$</span>
    </div>

    <header class="topbar">
      <a class="brand" href="#top" aria-label="Signalume home">
        <span class="brand-mark" aria-hidden="true"><i></i><i></i><i></i><i></i></span>
        <span class="brand-word">SIGNALUME</span>
      </a>
      <nav class="nav-links" aria-label="Primary navigation">
        <a class="active" href="#markets">Markets</a>
        <a href="#portfolio">Portfolio</a>
        <a href="#how-it-works">How it works</a>
      </nav>
      <div class="top-actions">
        <span class="chain-chip"><span class="chain-dot"></span> Robinhood Chain</span>
        <button class="wallet-button" type="button" @click="walletOpen = true">
          <span class="wallet-icon">[ ]</span>{{ connected ? '0x7A...BEEF' : 'Connect wallet' }}
        </button>
      </div>
    </header>

    <main id="top">
      <section class="hero" aria-labelledby="hero-title">
        <div class="hero-copy">
          <div class="eyebrow"><span class="blink-dot"></span> LIVE ON ROBINHOOD CHAIN</div>
          <h1 id="hero-title">PREDICT THE<br /><span>ODD</span> OUTCOME.</h1>
          <p>Trade the world's weirdest questions. Built for fast moves, tiny fees, and big <strong>1UPs.</strong></p>
          <div class="hero-buttons">
            <a class="primary-cta" href="#markets">Explore markets <span>-&gt;</span></a>
            <button class="text-cta" type="button" @click="walletOpen = true">Get started <span>-&gt;</span></button>
          </div>
        </div>
        <div class="hero-console" aria-label="Market stats">
          <div class="console-top"><span>SIGNALUME // TERMINAL 01</span><span class="signal">* ONLINE</span></div>
          <div class="console-screen">
            <div class="screen-line"><span>MARKETS LIVE</span><strong>128</strong></div>
            <div class="screen-line"><span>24H VOLUME</span><strong>$2.4M</strong></div>
            <div class="screen-line"><span>AVG. FEE</span><strong>0.2%</strong></div>
            <div class="mini-chart" aria-hidden="true"><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i><i></i></div>
          </div>
          <div class="console-foot"><span>RBH-01</span><span>INSERT COIN <b>*</b></span></div>
        </div>
      </section>

      <section id="markets" class="markets-section" aria-labelledby="markets-title">
        <div class="section-head">
          <div><div class="section-kicker">// CHOOSE YOUR LEVEL</div><h2 id="markets-title">MARKET MAP</h2></div>
          <div class="live-count"><span></span> 128 markets live</div>
        </div>
        <div class="market-controls">
          <div class="tabs" role="tablist" aria-label="Market categories">
            <button v-for="tab in tabs" :key="tab" type="button" :class="{ selected: activeTab === tab }" @click="activeTab = tab">{{ tab }}</button>
          </div>
          <label class="search-box"><span aria-hidden="true">?</span><input v-model="query" type="search" placeholder="Search markets" aria-label="Search markets" /></label>
        </div>

        <div class="market-list">
          <div class="market-list-head"><span>MARKET</span><span>ODDS</span><span>24H MOVE</span><span>VOLUME</span><span></span></div>
          <article v-for="market in filteredMarkets" :key="market.id" class="market-row">
            <div class="market-name"><div :class="['pixel-icon', `icon-${market.icon}`]" aria-hidden="true"><span></span></div><div><h3>{{ market.title }}</h3><p>{{ market.subtitle }} <em v-if="market.hot">HOT</em></p></div></div>
            <div class="odds"><strong>{{ market.yes }}%</strong><span>YES</span></div>
            <div :class="['move', market.change.startsWith('-') ? 'down' : 'up']">{{ market.change }}</div>
            <div class="volume">{{ market.volume }}</div>
            <div class="row-actions"><button type="button" class="yes-button" @click="openTrade(market, 'YES')">YES</button><button type="button" class="no-button" @click="openTrade(market, 'NO')">NO</button></div>
          </article>
          <div v-if="!filteredMarkets.length" class="empty-state">No markets found in this level.</div>
        </div>
      </section>

      <section id="portfolio" class="bottom-band">
        <div class="ticker-title"><span class="coin-stack">$</span><div><span class="section-kicker">// YOUR RUN</span><h2>PORTFOLIO</h2></div></div>
        <div class="portfolio-stat"><span>AVAILABLE BALANCE</span><strong>{{ connected ? '$1,240.00' : '---' }}</strong></div>
        <div class="portfolio-stat"><span>OPEN POSITIONS</span><strong>{{ connected ? '03' : '00' }}</strong></div>
        <button class="outline-button" type="button" @click="walletOpen = true">{{ connected ? 'View portfolio ->' : 'Connect to view ->' }}</button>
      </section>

      <section id="how-it-works" class="how-section"><div class="section-kicker">// THREE EASY MOVES</div><h2>PLAY THE ODDS.</h2><div class="steps"><div><b>01</b><h3>Pick a question</h3><p>Find a market where your read is stronger than the crowd.</p></div><div><b>02</b><h3>Choose YES or NO</h3><p>Back your call with USDC on Robinhood Chain.</p></div><div><b>03</b><h3>Collect your 1UP</h3><p>When the outcome lands, winners split the pool.</p></div></div></section>
    </main>

    <footer class="footer"><span>SIGNALUME / 2025</span><span>BUILT ON <a :href="contractExplorerUrl" target="_blank" rel="noreferrer"><b>ROBINHOOD CHAIN</b></a> // {{ contractAddress.slice(0, 6) }}...{{ contractAddress.slice(-4) }}</span><span>CAUTION: OUTCOMES MAY BE ODD.</span></footer>

    <div v-if="selectedMarket" class="modal-backdrop" @click.self="closeTrade">
      <section class="trade-modal" role="dialog" aria-modal="true" aria-labelledby="trade-title">
        <button class="close-button" type="button" aria-label="Close trade dialog" @click="closeTrade">X</button>
        <div class="modal-kicker">LEVEL {{ String(selectedMarket.id).padStart(2, '0') }} // TRADE</div>
        <h2 id="trade-title">{{ selectedMarket.title }}</h2><p class="modal-subtitle">{{ selectedMarket.subtitle }}</p>
        <div class="side-switch"><button type="button" :class="{ active: selectedSide === 'YES' }" @click="selectedSide = 'YES'">YES <strong>{{ selectedMarket.yes }}%</strong></button><button type="button" :class="{ active: selectedSide === 'NO' }" @click="selectedSide = 'NO'">NO <strong>{{ 100 - selectedMarket.yes }}%</strong></button></div>
        <label class="amount-label">AMOUNT <span>USDC</span><input v-model="stake" type="number" min="1" step="1" /></label>
        <div class="payout-line"><span>Potential payout</span><strong>${{ payout }}</strong></div>
        <button class="confirm-button" type="button" @click="confirmTrade">{{ connected ? 'Confirm position' : 'Connect wallet to trade' }} <span>-&gt;</span></button>
        <p class="modal-note">Trades settle automatically when the market resolves.</p>
      </section>
    </div>

    <div v-if="walletOpen" class="modal-backdrop" @click.self="walletOpen = false"><section class="wallet-modal" role="dialog" aria-modal="true" aria-labelledby="wallet-title"><button class="close-button" type="button" aria-label="Close wallet dialog" @click="walletOpen = false">X</button><div class="wallet-pixel">*</div><div class="modal-kicker">ROBINHOOD CHAIN</div><h2 id="wallet-title">CONNECT TO PLAY</h2><p>Use a compatible wallet to trade markets with USDC.</p><button class="confirm-button" type="button" @click="connectWallet">Connect wallet <span>-&gt;</span></button><p class="modal-note">By connecting, you agree to the Signalume terms.</p></section></div>
    <transition name="toast"><div v-if="toast" class="toast">{{ toast }}</div></transition>
  </div>
</template>
