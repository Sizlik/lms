<template>
	<div>
		<!-- Filters Section -->
		<div class="mb-6 flex flex-col space-y-4 sm:space-y-0 sm:flex-row sm:items-center sm:justify-between">
			<div class="flex items-center gap-3">
				<div class="flex items-center gap-2">
					<div class="w-1 h-8 bg-blue-600 dark:bg-blue-500 rounded-full"></div>
					<h2 class="text-xl font-bold text-ink-gray-9 dark:text-white">
						{{ __('Расписание') }}
					</h2>
				</div>
			</div>
			
			<div class="flex flex-col sm:flex-row gap-3 items-stretch sm:items-center">
				<!-- Category Filter -->
				<div class="relative group">
					<div class="absolute left-3 top-1/2 -translate-y-1/2 pointer-events-none z-10">
						<Layers class="w-4 h-4 text-gray-400 dark:text-gray-500 group-hover:text-blue-600 dark:group-hover:text-blue-400 transition-colors" />
					</div>
					<Select
						v-model="selectedCategory"
						:options="categoryOptions"
						:placeholder="__('Категория')"
						class="w-full sm:w-56 pl-10"
						@update:modelValue="onCategoryChange"
					/>
				</div>

				<!-- Instructor Filter -->
				<div class="relative group">
					<div class="absolute left-3 top-1/2 -translate-y-1/2 pointer-events-none z-10">
						<UserCircle class="w-4 h-4 text-gray-400 dark:text-gray-500 group-hover:text-blue-600 dark:group-hover:text-blue-400 transition-colors" />
					</div>
					<Select
						v-model="selectedInstructor"
						:options="instructorOptions"
						:placeholder="__('Преподаватель')"
						class="w-full sm:w-56 pl-10"
						:disabled="instructorsLoading"
						@update:modelValue="onInstructorChange"
					/>
				</div>
			</div>
		</div>

		<!-- Active Filters Display -->
		<div v-if="selectedCategory || selectedInstructor" class="mb-4 flex flex-wrap gap-2">
			<div v-if="selectedCategory" class="inline-flex items-center gap-2 px-3 py-1.5 bg-blue-50 dark:bg-blue-900/20 text-blue-700 dark:text-blue-300 rounded-full text-sm font-medium">
				<Layers class="w-3.5 h-3.5" />
				<span>{{ selectedCategory }}</span>
				<button @click="selectedCategory = null; onCategoryChange()" class="hover:bg-blue-100 dark:hover:bg-blue-800/40 rounded-full p-0.5 transition-colors">
					<X class="w-3.5 h-3.5" />
				</button>
			</div>
			
			<div v-if="selectedInstructor" class="inline-flex items-center gap-2 px-3 py-1.5 bg-green-50 dark:bg-green-900/20 text-green-700 dark:text-green-300 rounded-full text-sm font-medium">
				<UserCircle class="w-3.5 h-3.5" />
				<span>{{ getInstructorName(selectedInstructor) }}</span>
				<button @click="selectedInstructor = null; onInstructorChange()" class="hover:bg-green-100 dark:hover:bg-green-800/40 rounded-full p-0.5 transition-colors">
					<X class="w-3.5 h-3.5" />
				</button>
			</div>
			
			<button 
				@click="clearAllFilters"
				class="inline-flex items-center gap-1.5 px-3 py-1.5 text-gray-600 dark:text-gray-400 hover:text-gray-900 dark:hover:text-gray-200 text-sm font-medium transition-colors"
			>
				<span>{{ __('Очистить все') }}</span>
			</button>
		</div>

		<CalendarSchedule
			:category-filter="selectedCategory"
			:instructor-filter="selectedInstructor"
		/>
	</div>
</template>
<script setup lang="ts">
import { inject, ref, computed, watch, onMounted } from 'vue'
import { createResource, createListResource, Select } from 'frappe-ui'
import { formatTime } from '@/utils'
import {
	Calendar,
	Clock,
	Info,
	Monitor,
	MoveRight,
	Video,
	Layers,
	UserCircle,
	X,
} from 'lucide-vue-next'
import CourseCard from '@/components/CourseCard.vue'
import BatchCard from '@/components/BatchCard.vue'
import UpcomingEvaluations from '@/components/UpcomingEvaluations.vue'
import CalendarSchedule from '@/components/CalendarSchedule.vue'

const dayjs = inject<any>('$dayjs')
const user = inject<any>('$user')

const props = defineProps<{
	myLiveClasses: any
}>()

const selectedCategory = ref(null)
const selectedInstructor = ref(null)

const myCourses = createResource({
	url: 'lms.lms.api.get_my_courses',
	auto: true,
})

const myBatches = createResource({
	url: 'lms.lms.api.get_my_batches',
	auto: true,
})

// Get categories from backend
const categoriesList = createListResource({
	doctype: 'LMS Category',
	fields: ['name', 'category'],
	auto: true,
	orderBy: 'category asc',
})

// Get all instructors from backend
const instructorsList = createListResource({
	doctype: 'Course Evaluator',
	fields: ['evaluator', 'full_name', 'username'],
	auto: true,
	orderBy: 'full_name asc',
})

// Get batches for filtering instructors by category
const batchesForInstructors = createListResource({
	doctype: 'LMS Batch',
	url: 'lms.lms.utils.get_batches',
	filters: {},
	fields: ['instructors', 'categories'],
	auto: true,
})

const categoryOptions = computed(() => {
	const options = [{ label: __('Все категории'), value: null }]
	if (categoriesList.data) {
		categoriesList.data.forEach((cat) => {
			if (cat.category) {
				options.push({ label: cat.category, value: cat.category })
			}
		})
	}
	return options
})

const instructorOptions = computed(() => {
	const options = [{ label: __('Все преподаватели'), value: null }]
	
	if (!instructorsList.data) return options
	
	// If category is selected (not null), filter instructors based on batches
	if (selectedCategory.value) {
		if (!batchesForInstructors.data) return options
		
		const instructorSet = new Set()
		
		batchesForInstructors.data.forEach((batch) => {
			// Check if batch has the selected category
			const hasCategory = batch.categories && 
				batch.categories.some((cat) => cat.category === selectedCategory.value)
			
			if (hasCategory && batch.instructors) {
				batch.instructors.forEach((inst) => {
					const instructorName = inst.instructor || inst.name
					if (instructorName) {
						instructorSet.add(instructorName)
					}
				})
			}
		})
		
		// Filter instructors list to only show those in the category
		instructorsList.data.forEach((inst) => {
			if (instructorSet.has(inst.evaluator)) {
				options.push({
					label: inst.full_name || inst.evaluator,
					value: inst.evaluator
				})
			}
		})
	} else {
		// No category selected (null), show all instructors
		instructorsList.data.forEach((inst) => {
			options.push({
				label: inst.full_name || inst.evaluator,
				value: inst.evaluator
			})
		})
	}
	
	return options
})

const instructorsLoading = computed(() => {
	return instructorsList.list?.loading || batchesForInstructors.list?.loading
})

const onCategoryChange = () => {
	// Don't reset instructor filter when category changes
	// Reload batches with or without category filter
	if (selectedCategory.value) {
		// Load batches for the selected category to get instructors
		batchesForInstructors.update({
			filters: { category: selectedCategory.value }
		})
	} else {
		// Load all batches when "Все категории" is selected
		batchesForInstructors.update({
			filters: {}
		})
	}
	batchesForInstructors.reload()
}

const onInstructorChange = () => {
	// Instructor changed, calendar will update automatically
}

const clearAllFilters = () => {
	selectedCategory.value = null
	selectedInstructor.value = null
	onCategoryChange()
}

const getInstructorName = (evaluator) => {
	if (!instructorsList.data) return evaluator
	const instructor = instructorsList.data.find(inst => inst.evaluator === evaluator)
	return instructor ? (instructor.full_name || instructor.evaluator) : evaluator
}

const getClassEnd = (cls: { date: string; time: string; duration: number }) => {
	const classStart = new Date(`${cls.date}T${cls.time}`)
	return new Date(classStart.getTime() + cls.duration * 60000)
}

const canAccessClass = (cls: {
	date: string
	time: string
	duration: number
}) => {
	if (cls.date < dayjs().format('YYYY-MM-DD')) return false
	if (cls.date > dayjs().format('YYYY-MM-DD')) return false
	if (hasClassEnded(cls)) return false
	return true
}

const hasClassEnded = (cls: {
	date: string
	time: string
	duration: number
}) => {
	const classEnd = getClassEnd(cls)
	const now = new Date()
	return now > classEnd
}
</script>
