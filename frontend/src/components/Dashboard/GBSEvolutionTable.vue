<template>
  <div class="mx-5 my-4 flex flex-col gap-4">
    <section
      v-for="table in tables"
      :key="table.key"
      class="rounded-lg border border-outline-gray-2 bg-surface-white"
    >
      <div class="flex flex-wrap items-center justify-between gap-3 border-b border-outline-gray-2 px-4 py-3">
        <div>
          <h2 class="text-base font-semibold text-ink-gray-9">{{ table.title }}</h2>
          <p class="text-sm text-ink-gray-5">
            24 h par date de création ; 7 j, 30 j et 6 mois par date de demande
          </p>
        </div>
        <Button
          variant="outline"
          :loading="table.resource.loading"
          @click="table.resource.reload()"
        >
          Actualiser
        </Button>
      </div>

      <div v-if="table.resource.loading" class="p-6 text-sm text-ink-gray-5">
        Chargement des statistiques…
      </div>
      <div v-else-if="table.resource.error" role="alert" class="p-6 text-sm text-ink-red-6">
        Impossible de charger le rapport.
      </div>
      <div v-else class="max-h-[440px] overflow-auto">
        <table class="w-full min-w-[920px] border-collapse text-sm" :aria-label="table.title">
          <thead class="sticky top-0 z-10 bg-surface-gray-1">
            <tr class="border-b border-outline-gray-2 text-left text-ink-gray-7">
              <th scope="col" class="px-4 py-3">{{ table.firstLabel }}</th>
              <th scope="col" class="px-3 py-3 text-right">24 h</th>
              <th scope="col" class="px-3 py-3 text-right">7 jours</th>
              <th scope="col" class="px-3 py-3 text-right">Évol. 7j</th>
              <th scope="col" class="px-3 py-3 text-right">30 jours</th>
              <th scope="col" class="px-3 py-3 text-right">Évol. 30j</th>
              <th scope="col" class="px-3 py-3 text-right">6 mois</th>
              <th scope="col" class="px-4 py-3 text-right">Total</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(row, index) in tableRows(table.resource)"
              :key="String(row[table.key])"
              class="border-b border-outline-gray-1"
              :class="index === 0 ? 'bg-surface-gray-1 font-semibold' : ''"
            >
              <th scope="row" class="whitespace-nowrap px-4 py-3 text-left">
                {{ row[table.key] }}
              </th>
              <td class="px-3 py-3 text-right tabular-nums">{{ numeric(row.h24) }}</td>
              <td class="px-3 py-3 text-right tabular-nums">{{ numeric(row.j7) }}</td>
              <td class="px-3 py-3 text-right tabular-nums" :class="growthClass(row.var7)">
                {{ row.var7 }}
              </td>
              <td class="px-3 py-3 text-right tabular-nums">{{ numeric(row.j30) }}</td>
              <td class="px-3 py-3 text-right tabular-nums" :class="growthClass(row.var30)">
                {{ row.var30 }}
              </td>
              <td class="px-3 py-3 text-right tabular-nums">{{ numeric(row.m6) }}</td>
              <td class="px-4 py-3 text-right tabular-nums">{{ numeric(row.total) }}</td>
            </tr>
            <tr v-if="tableRows(table.resource).length === 0">
              <td colspan="8" class="px-4 py-6 text-center text-ink-gray-5">
                Aucune donnée autorisée.
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>
  </div>
</template>

<script setup lang="ts">
import { Button, createResource } from 'frappe-ui'

type EvolutionRow = {
  marque?: string
  site_source?: string
  h24: number
  j7: number
  var7: string
  j30: number
  var30: string
  m6: number
  total: number
  [key: string]: number | string | undefined
}

type ReportResource = {
  data?: { rows?: EvolutionRow[] }
  loading?: boolean
  error?: unknown
  reload: () => void
}

const brandReport = createResource({
  url: 'gbs_crm_custom.api.evolution_report.get_report',
  auto: true,
})
const sourceReport = createResource({
  url: 'gbs_crm_custom.api.source_report.get_report',
  auto: true,
})

const tables = [
  {
    key: 'marque',
    firstLabel: 'Marque',
    title: 'Prospects par marque — Évolution',
    resource: brandReport as ReportResource,
  },
  {
    key: 'site_source',
    firstLabel: 'Site source',
    title: 'Prospects par site source — Évolution',
    resource: sourceReport as ReportResource,
  },
]

const tableRows = (resource: ReportResource) =>
  Array.isArray(resource.data?.rows) ? resource.data.rows : []
const numeric = (value: unknown) => Number(value || 0).toLocaleString('fr-FR')
const growthClass = (value: unknown) => {
  if (typeof value !== 'string') return ''
  if (value.startsWith('+')) return 'text-ink-green-2'
  if (value.startsWith('-')) return 'text-ink-red-3'
  return ''
}
</script>
