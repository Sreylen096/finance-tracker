<template>
    <section class="flex items-center justify-between mb-10">
        <p class="text-4xl font-extrabold ">
            Summary
        </p>
        <div>
            <USelectMenu :options="transactionViewOptions" v-model="selectedView" />
        </div>
    </section>

    <section class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 sm:gap-16 mb-10">
        <Trend color="green" title="Income" :amount="4000" :last-amount="3000" :loading="false" />
        <Trend color="red" title="Expense" :amount="4000" :last-amount="5000" :loading="false" />
        <Trend color="green" title="Investments" :amount="4000" :last-amount="3000" :loading="false" />
        <Trend color="red" title="Saving" :amount="4000" :last-amount="4100" :loading="false" />
    </section>
    <section class="flex justify-between mb-10">
        <div>
            <h2 class="text-2xl font-extrabold">Transactions</h2>
            <div class="text-gray-500 dark:text-gray-400">
                You have {{ incomeCount }} incomes and {{ expenseCount }} expenses this period
            </div>
        </div>
        <div>
            <UButton icon="i-heroicons-plus-circle" color="white" variant="solid" label="Add" />
        </div>
    </section>
    <section>
        <div v-for="(transactionsOnDay, date) in transactionsGroupedByDate" :key="date" class="mb-10">
            <DailyTransactionSummary :date="date" :transactions="transactionsOnDay" />
            <Transaction v-for="transaction in transactionsOnDay" :key="transaction.id" :transaction="transaction" />
        </div>
    </section>
</template>

<script setup>
import { transactionViewOptions } from '#imports'
const supabase = useSupabaseClient()

const selectedView = ref(transactionViewOptions[1])
const transactions = ref([])

const income = computed(
    () => transactions.value.filter(t => t.type === 'Income')
)
const expense = computed(
    () => transactions.value.filter(t => t.type === 'Expense')
)
const incomeCount = computed(() => income.value.length)
const expenseCount = computed(() => expense.value.length)

const incomeTotal = computed(
    () => income.value.reduce((sum, transaction) => sum + transaction.amount, 0)
)
const expenseTotal = computed(
    () => expense.value.reduce((sum, transaction) => sum + transaction.amount, 0)
)

const { data, pending } = await useAsyncData('transactions', async () => {
    const { data, error } = await supabase
        .from('transactions')
        .select()
    if (error) return []

    return data
})
console.log('AsyncData result:', data.value)
transactions.value = data.value

const transactionsGroupedByDate = computed(() => {
    let grouped = {}

    for (const transaction of transactions.value) {
        const date = new Date(transaction.created_at).toISOString().split('T')[0]

        if (!grouped[date]) {
            grouped[date] = []
        }

        grouped[date].push(transaction)
    }

    return grouped
})

console.log(transactionsGroupedByDate.value)
</script>
