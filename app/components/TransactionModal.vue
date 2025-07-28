<template>
    <UModal :dismissible="false" title="Modal with description" v-model="isOpen"
        description="Lorem ipsum dolor sit amet, consectetur adipiscing elit.">

        <!-- Add button (optional, can remove or trigger modal submission) -->
        <UButton icon="i-heroicons-plus-circle" label="Add" color="neutral" variant="subtle" />

        <!-- Form wrapped in UCard -->
        <template #body>
            <UCard>
                <UForm class="space-y-4" :state="state" :schema="schema" ref="form" @submit.prevent="save">
                    <!-- Transaction Type -->
                    <UFormField label="Transaction Type" name="type" required>
                        <USelect v-model="state.type" placeholder="Select the transaction type"
                            class="w-full h-[40px] !z-100" :items="types" />
                    </UFormField>

                    <!-- Amount -->
                    <UFormField label="Amount" name="amount" required>
                        <UInput v-model="state.amount" type="number" placeholder="Amount" class="w-full h-[40px]" />
                    </UFormField>

                    <!-- Transaction Date -->
                    <UFormField label="Transaction Date" name="created_at" required>
                        <UInput v-model="state.created_at" type="date" icon="i-heroicons-calendar-days-20-solid"
                            class="w-full h-[40px]" />
                    </UFormField>

                    <!-- Description -->
                    <UFormField label="Description" name="description" hint="Optional">
                        <UInput v-model="state.description" placeholder="Write a short description"
                            class="w-full h-[40px]" />
                    </UFormField>

                    <UFormField label="Category" name="category" :required="true" v-if="state.type === 'Expense'">
                        <USelect v-model="state.category" placeholder="Select the category" class="w-full h-[40px]"
                            :items="categories" />
                    </UFormField>

                    <!-- Submit Button -->
                    <div class="pt-2">
                        <UButton type="submit" label="Submit" color="neutral" :loading="isLoading" />
                    </div>
                </UForm>
            </UCard>
        </template>
    </UModal>
</template>

<script setup>
import { categories, types } from '@/utils/constant'
import { z } from 'zod'

const isLoading = ref(false)
const supabase = useSupabaseClient()
const toast = useToast()
const defaultSchema = z.object({
    created_at: z.string(),
    description: z.string().optional(),
    amount: z.number().positive('Amount needs to be more than 0')
})

const emit = defineEmits(['update:modelValue', 'saved'])
const incomeSchema = z.object({
    type: z.literal('Income')
})
const expenseSchema = z.object({
    type: z.literal('Expense'),
    category: z.enum(categories)
})
const investmentSchema = z.object({
    type: z.literal('Investment')
})
const savingSchema = z.object({
    type: z.literal('Saving')
})

const schema = z.intersection(
    z.discriminatedUnion('type', [incomeSchema, expenseSchema, investmentSchema, savingSchema]),
    defaultSchema
)
const props = defineProps({
  modelValue: Boolean
})
const form = ref()

const save = async () => {
    if (form.value.errors.length) return

    isLoading.value = true
    try {
        const { error } = await supabase.from('transactions')
            .upsert({ ...state.value })

        if (!error) {
            toast.add({
                'title': 'Transaction saved',
                'icon': 'i-heroicons-check-circle'
            })
            isOpen.value = false
            emit('saved')
            return
        }

        throw error
    } catch (e) {
        toast.add({
            title: 'Transaction not saved',
            description: e.message,
            icon: 'i-heroicons-exclamation-circle',
            color: 'red'
        })
    } finally {
        isLoading.value = false
    }
}

const initialState = {
    type: undefined,
    amount: 0,
    created_at: undefined,
    description: undefined,
    category: undefined
}
const state = ref({
    ...initialState
})
const resetForm = () => {
    Object.assign(state.value, initialState)
    form.value.clear()
}

const isOpen = computed({
    get: () => props.modelValue,
    set: (value) => {
        if (!value) resetForm()
        emit('update:modelValue', value)
    }
})
</script>
