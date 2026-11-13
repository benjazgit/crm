<template>
  <section class="flex h-full min-h-0 w-full flex-col overflow-hidden rounded-lg border border-outline-gray-2 bg-surface-white">
    <div class="flex items-center justify-between gap-2 px-4 py-3">
      <div>
        <h2 class="text-sm font-semibold text-ink-gray-9">Suivi commercial</h2>
        <p class="text-xs text-ink-gray-5">Prospects créés sur la période sélectionnée · périmètre autorisé</p>
      </div>
      <Button variant="outline" :loading="report.loading" @click="report.reload()">Actualiser</Button>
    </div>
    <div v-if="report.loading" class="px-4 py-5 text-sm text-ink-gray-5">Chargement…</div>
    <div v-else-if="report.error" role="alert" class="px-4 py-5 text-sm text-ink-red-6">Impossible de charger le suivi commercial.</div>
    <div v-else class="min-h-0 flex-1 overflow-auto">
      <table class="w-full min-w-[670px] border-collapse text-xs" aria-label="Suivi commercial">
        <thead class="sticky top-0 z-10 bg-surface-gray-1">
          <tr class="border-y border-outline-gray-2 text-ink-gray-7">
            <th scope="col" class="px-3 py-2 text-left">Commercial</th>
            <th scope="col" class="px-2 py-2 text-right">Prospects</th>
            <th scope="col" class="px-2 py-2 text-right">En cours</th>
            <th scope="col" class="px-2 py-2 text-right">Convertis</th>
            <th scope="col" class="px-2 py-2 text-right">Perdus</th>
            <th scope="col" class="px-3 py-2 text-right">Conversion</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="row in rows" :key="row.user" class="border-b border-outline-gray-1">
            <th scope="row" class="px-3 py-2 text-left">{{ row.commercial }}</th>
            <td class="px-2 py-2 text-right tabular-nums">{{ num(row.prospects) }}</td>
            <td class="px-2 py-2 text-right tabular-nums">{{ num(row.en_cours) }}</td>
            <td class="px-2 py-2 text-right tabular-nums">{{ num(row.convertis) }}</td>
            <td class="px-2 py-2 text-right tabular-nums">{{ num(row.perdus) }}</td>
            <td class="px-3 py-2 text-right tabular-nums">{{ rate(row) }}</td>
          </tr>
          <tr v-if="!rows.length">
            <td colspan="6" class="px-3 py-4 text-center text-ink-gray-5">Aucun prospect sur la période.</td>
          </tr>
        </tbody>
      </table>
    </div>
  </section>
</template>

<script setup lang="ts">
import { computed, inject, watch, type Ref } from 'vue'
import { Button, createResource } from 'frappe-ui'
type Row = {
  user: string
  commercial: string
  prospects: number
  en_cours: number
  convertis: number
  perdus: number
}
const fromDate = inject<Ref<string | null>>('fromDate')
const toDate = inject<Ref<string | null>>('toDate')
const report = createResource({
  url: 'gbs_crm_custom.api.commercial_report.get_report',
  makeParams() {
    return { from_date: fromDate?.value || null, to_date: toDate?.value || null }
  },
  auto: true,
})
watch([() => fromDate?.value, () => toDate?.value], () => report.reload())
const rows = computed<Row[]>(() => Array.isArray(report.data?.rows) ? report.data.rows : [])
const num = (value: number) => Number(value || 0).toLocaleString('fr-FR')
const rate = (row: Row) => row.prospects ? Math.round(100 * row.convertis / row.prospects) + ' %' : '0 %'
</script>
