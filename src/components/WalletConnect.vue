
<template>
  <div class="space-y-4">
    <select v-model="selected" class="border rounded px-3 py-2">
      <option disabled value="">Pilih Chain</option>
      <option v-for="(data, key) in chains" :key="key" :value="key">
        {{ chains[key].chainName }}
      </option>
    </select>
    <button @click="connect" class="bg-blue-600 text-white px-4 py-2 rounded">
      {{ connected ? 'Connected: ' + address : 'Connect Wallet' }}
    </button>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { CHAINS } from '../chains/chains'

const selected = ref('helios')
const address = ref('')
const connected = ref(false)
const chains = CHAINS

const connect = async () => {
  try {
    if (!window.keplr || !selected.value) {
      alert('Keplr wallet not found or chain not selected.')
      return
    }
    const chain = chains[selected.value]
    await window.keplr.experimentalSuggestChain({
      chainId: chain.chainId.toString(),
      chainName: chain.chainName,
      rpc: chain.rpc,
      rest: chain.rpc,
      bip44: { coinType: 118 },
      bech32Config: {
        bech32PrefixAccAddr: "cosmos",
        bech32PrefixAccPub: "cosmospub",
        bech32PrefixValAddr: "cosmosvaloper",
        bech32PrefixValPub: "cosmosvaloperpub",
        bech32PrefixConsAddr: "cosmosvalcons",
        bech32PrefixConsPub: "cosmosvalconspub"
      },
      currencies: [chain.currency],
      feeCurrencies: [chain.currency],
      stakeCurrency: chain.currency,
      gasPriceStep: {
        low: 0.01,
        average: 0.025,
        high: 0.04
      },
      features: ["stargate", "ibc-transfer"]
    })
    await window.keplr.enable(chain.chainId.toString())
    const offlineSigner = window.getOfflineSigner(chain.chainId.toString())
    const accounts = await offlineSigner.getAccounts()
    address.value = accounts[0].address
    connected.value = true
  } catch (err) {
    console.error(err)
    alert('Failed to connect wallet')
  }
}
</script>
