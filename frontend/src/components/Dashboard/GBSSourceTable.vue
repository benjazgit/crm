<template>
  <section class="flex h-full min-h-0 w-full flex-col overflow-hidden rounded-lg border border-outline-gray-2 bg-surface-white">
    <div class="flex items-center justify-between gap-2 px-4 py-3">
      <div class="min-w-0">
        <h2 class="text-sm font-semibold text-ink-gray-9">Prospects par site source — Évolution</h2>
        <p class="text-xs text-ink-gray-5">24 h : création ; autres périodes : date de demande</p>
      </div>
      <Button variant="outline" :loading="report.loading" @click="report.reload()">Actualiser</Button>
    </div>
    <div v-if="report.loading" class="px-4 py-5 text-sm text-ink-gray-5">Chargement…</div>
    <div v-else-if="report.error" role="alert" class="px-4 py-5 text-sm text-ink-red-6">Impossible de charger le tableau.</div>
    <div v-else class="min-h-0 flex-1 overflow-auto">
      <table class="w-full min-w-[780px] border-collapse text-xs" aria-label="Évolution des prospects par site source">
        <thead class="sticky top-0 z-10 bg-surface-gray-1">
          <tr class="border-y border-outline-gray-2 text-ink-gray-7">
            <th scope="col" class="px-3 py-2 text-left">Site source</th>
            <th scope="col" class="px-2 py-2 text-right">24 h</th>
            <th scope="col" class="px-2 py-2 text-right">7 j</th>
            <th scope="col" class="px-2 py-2 text-right">Évol. 7 j</th>
            <th scope="col" class="px-2 py-2 text-right">30 j</th>
            <th scope="col" class="px-2 py-2 text-right">Évol. 30 j</th>
            <th scope="col" class="px-2 py-2 text-right">6 mois</th>
            <th scope="col" class="px-3 py-2 text-right">Total</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, i) in rows" :key="row.site_source" class="border-b border-outline-gray-1" :class="i === 0 ? 'bg-surface-gray-1 font-semibold' : ''">
            <th scope="row" class="px-3 py-2 text-left">{{ row.site_source }}</th>
            <td class="px-2 py-2 text-right tabular-nums">{{ num(row.h24) }}</td>
            <td class="px-2 py-2 text-right tabular-nums">{{ num(row.j7) }}</td>
            <td class="px-2 py-2 text-right tabular-nums" :class="growth(row.var7)">{{ row.var7 }}</td>
            <td class="px-2 py-2 text-right tabular-nums">{{ num(row.j30) }}</td>
            <td class="px-2 py-2 text-right tabular-nums" :class="growth(row.var30)">{{ row.var30 }}</td>
            <td class="px-2 py-2 text-right tabular-nums">{{ num(row.m6) }}</td>
            <td class="px-3 py-2 text-right tabular-nums">{{ num(row.total) }}</td>
          </tr>
          <tr v-if="!rows.length"><td colspan="8" class="px-3 py-4 text-center text-ink-gray-5">Aucune donnée autorisée.</td></tr>
        </tbody>
      </table>
    </div>
  </section>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { Button, createResource } from 'frappe-ui'

type Row = {
  site_source: string
  h24: number
  j7: number
  var7: string
  j30: number
  var30: string
  m6: number
  total: number
}
const report = createResource({
  url: 'gbs_crm_custom.api.source_report.get_report',
  auto: true,
})
const rows = computed<Row[]>(() => Array.isArray(report.data?.rows) ? report.data.rows : [])
const num = (v: number) => Number(v || 0).toLocaleString('fr-FR')
const growth = (v: string) => v?.startsWith('+') ? 'text-ink-green-2' : v?.startsWith('-') ? 'text-ink-red-3' : ''
</script>
