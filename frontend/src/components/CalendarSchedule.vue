<template>
	<div class="calendar-schedule dark:bg-gray-900 dark:text-gray-100">
		<div class="flex items-center justify-between mb-6 flex-wrap gap-4">
			<div class="flex items-center gap-4">
				<Button variant="ghost" @click="goToToday" class="text-sm dark:text-gray-300 dark:hover:bg-gray-800">
					{{ __('Today') }}
				</Button>
				<div class="flex items-center gap-2">
					<Button variant="ghost" @click="previousPeriod" class="p-2 dark:text-gray-300 dark:hover:bg-gray-800">
						<ChevronLeft class="h-4 w-4" />
					</Button>
					<Button variant="ghost" @click="nextPeriod" class="p-2 dark:text-gray-300 dark:hover:bg-gray-800">
						<ChevronRight class="h-4 w-4" />
					</Button>
				</div>
				<div class="text-lg font-semibold text-ink-gray-9 dark:text-white">
					{{ currentPeriodTitle }}
				</div>
			</div>
			<div class="flex items-center gap-2">
				<Button
					variant="solid"
					@click="showCreateBatchModal = true"
					class="dark:bg-blue-600 dark:hover:bg-blue-700"
				>
					<template #prefix>
						<Plus class="h-4 w-4" />
					</template>
					{{ __('Add') }}
				</Button>
				<div class="flex items-center border rounded-md dark:border-gray-700 bg-white dark:bg-gray-800">
					<Button
						variant="ghost"
						:class="[
              'rounded-r-none dark:text-gray-300 dark:hover:bg-gray-700',
              view === 'month' ? 'bg-surface-gray-2 dark:bg-gray-700 text-gray-900 dark:text-white font-medium' : ''
            ]"
						@click="view = 'month'"
					>
						{{ __('Month') }}
					</Button>
					<Button
						variant="ghost"
						:class="[
              'rounded-none border-x dark:border-gray-700 dark:text-gray-300 dark:hover:bg-gray-700',
              view === 'week' ? 'bg-surface-gray-2 dark:bg-gray-700 text-gray-900 dark:text-white font-medium' : ''
            ]"
						@click="view = 'week'"
					>
						{{ __('Week') }}
					</Button>
					<Button
						variant="ghost"
						:class="[
              'rounded-l-none dark:text-gray-300 dark:hover:bg-gray-700',
              view === 'day' ? 'bg-surface-gray-2 dark:bg-gray-700 text-gray-900 dark:text-white font-medium' : ''
            ]"
						@click="view = 'day'"
					>
						{{ __('Day') }}
					</Button>
				</div>
			</div>
		</div>
		<div class="calendar-container bg-white dark:bg-gray-900 border dark:border-gray-700 rounded-lg overflow-hidden shadow-sm">
			<div v-if="view === 'month'" class="month-view">
				<div class="grid grid-cols-7 border-b dark:border-gray-700 bg-surface-gray-1 dark:bg-gray-800">
					<div
						v-for="day in weekDays"
						:key="day"
						class="p-3 text-center text-sm font-semibold text-ink-gray-7 dark:text-gray-400 border-r dark:border-gray-700 last:border-r-0"
					>
						{{ day }}
					</div>
				</div>
				<div class="grid grid-cols-7">
					<div
						v-for="(date, index) in monthDates"
						:key="index"
						class="min-h-[120px] border-r border-b dark:border-gray-700 last:border-r-0 p-2 transition-colors"
						:class="{
              'bg-surface-gray-0 dark:bg-gray-950': !isCurrentMonth(date),
              'bg-white dark:bg-gray-900': isCurrentMonth(date),
              'bg-blue-50 dark:bg-blue-900/20': isToday(date)
            }"
					>
						<div
							class="text-sm font-semibold mb-1"
							:class="{
                'text-ink-gray-5 dark:text-gray-600': !isCurrentMonth(date),
                'text-ink-gray-9 dark:text-gray-200': isCurrentMonth(date),
                'text-blue-600 dark:text-blue-400': isToday(date)
              }"
						>
							{{ dayjs(date).format('D') }}
						</div>
						<div class="space-y-1">
							<div
								v-for="event in getEventsForDate(date)"
								:key="event.id"
								class="text-xs p-1.5 rounded shadow-sm cursor-pointer hover:brightness-110 truncate font-medium"
								:style="{ backgroundColor: event.color, color: event.textColor }"
								@click="openEventModal(event)"
							>
								{{ event.title }}
							</div>
						</div>
					</div>
				</div>
			</div>

			<div v-if="view === 'week'" class="week-view">
				<div class="grid grid-cols-8 border-b dark:border-gray-700">
					<div class="p-3 border-r dark:border-gray-700 bg-surface-gray-1 dark:bg-gray-800"></div>
					<div
						v-for="day in weekDates"
						:key="day"
						class="p-3 text-center border-r dark:border-gray-700 last:border-r-0 bg-surface-gray-1 dark:bg-gray-800"
					>
						<div class="text-sm font-semibold text-ink-gray-9 dark:text-gray-300">
							{{ dayjs(day).format('ddd') }}
						</div>
						<div
							class="text-lg font-semibold mt-1"
							:class="isToday(day) ? 'text-blue-600 dark:text-blue-400' : 'text-ink-gray-9 dark:text-white'"
						>
							{{ dayjs(day).format('D') }}
						</div>
					</div>
				</div>
				<div class="grid grid-cols-8">
					<div class="border-r dark:border-gray-700 bg-surface-gray-1 dark:bg-gray-800">
						<div
							v-for="hour in hours"
							:key="hour"
							class="h-16 border-b dark:border-gray-700 p-2 text-xs text-ink-gray-5 dark:text-gray-400"
						>
							{{ hour }}
						</div>
					</div>
					<div
						v-for="day in weekDates"
						:key="day"
						class="border-r dark:border-gray-700 last:border-r-0 relative bg-white dark:bg-gray-900"
					>
						<div
							v-for="hour in hours"
							:key="hour"
							class="h-16 border-b border-dashed border-outline-gray-2 dark:border-gray-800"
						></div>
						<div
							v-for="event in getEventsForDate(day)"
							:key="event.id"
							class="absolute left-1 right-1 rounded p-1.5 text-xs cursor-pointer hover:brightness-110 z-10 shadow-sm border border-white/10"
							:style="{
                backgroundColor: event.color,
                color: event.textColor,
                top: getEventPosition(event, day) + '%',
                height: getEventHeight(event) + '%'
              }"
							@click="openEventModal(event)"
						>
							<div class="font-bold truncate">{{ event.title }}</div>
							<div class="text-xs opacity-90">{{ event.time }}</div>
						</div>
					</div>
				</div>
			</div>

			<div v-if="view === 'day'" class="day-view">
				<div class="grid grid-cols-2 border-b dark:border-gray-700">
					<div class="p-3 border-r dark:border-gray-700 bg-surface-gray-1 dark:bg-gray-800"></div>
					<div class="p-3 text-center bg-surface-gray-1 dark:bg-gray-800">
						<div class="text-sm font-semibold text-ink-gray-9 dark:text-gray-300">
							{{ dayjs(currentDate).format('dddd') }}
						</div>
						<div
							class="text-lg font-semibold mt-1"
							:class="isToday(currentDate) ? 'text-blue-600 dark:text-blue-400' : 'text-ink-gray-9 dark:text-white'"
						>
							{{ dayjs(currentDate).format('D MMMM YYYY') }}
						</div>
					</div>
				</div>
				<div class="grid grid-cols-2 bg-white dark:bg-gray-900">
					<div class="border-r dark:border-gray-700 bg-surface-gray-1 dark:bg-gray-800">
						<div
							v-for="hour in hours"
							:key="hour"
							class="h-16 border-b dark:border-gray-700 p-2 text-xs text-ink-gray-5 dark:text-gray-400"
						>
							{{ hour }}
						</div>
					</div>
					<div class="relative">
						<div
							v-for="hour in hours"
							:key="hour"
							class="h-16 border-b border-dashed border-outline-gray-2 dark:border-gray-800"
						></div>
						<div
							v-for="event in getEventsForDate(currentDate)"
							:key="event.id"
							class="absolute left-2 right-2 rounded p-3 cursor-pointer hover:brightness-110 z-10 shadow border border-white/10"
							:style="{
                backgroundColor: event.color,
                color: event.textColor,
                top: getEventPosition(event, currentDate) + '%',
                height: getEventHeight(event) + '%'
              }"
							@click="openEventModal(event)"
						>
							<div class="font-bold text-base">{{ event.title }}</div>
							<div class="text-sm mt-1 opacity-90 flex items-center gap-1">
								<Clock class="w-3 h-3" /> {{ event.time }}
							</div>
							<div v-if="event.description" class="text-xs mt-2 opacity-90 line-clamp-2">
								{{ event.description }}
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>

		<Dialog
			v-model="showCreateBatchModal"
			:options="{
				title: __('Создать занятие'),
				size: 'lg',
				actions: [
					{
						label: __('Create'),
						variant: 'solid',
						onClick: (close) => createBatch(close),
					},
				],
			}"
		>
			<template #body-content>
				<div class="space-y-4">
					<FormControl
						v-model="newBatch.title"
						:label="__('Title')"
						:required="true"
						class="w-full"
					/>
					<div class="grid grid-cols-1 md:grid-cols-2 gap-4">
						<FormControl
							v-model="newBatch.start_date"
							:label="__('Start Date')"
							type="date"
							:required="true"
						/>
						<FormControl
							v-model="newBatch.end_date"
							:label="__('End Date')"
							type="date"
							:required="true"
						/>
					</div>
					<div class="grid grid-cols-1 md:grid-cols-2 gap-4">
						<FormControl
							v-model="newBatch.start_time"
							:label="__('Start Time')"
							type="time"
							:required="true"
						/>
						<FormControl
							v-model="newBatch.end_time"
							:label="__('End Time')"
							type="time"
							:required="true"
						/>
					</div>
					<FormControl
						v-model="newBatch.description"
						:label="__('Description')"
						type="textarea"
						:rows="4"
						:required="true"
						:placeholder="__('Short description of the batch')"
					/>
					<div>
						<label class="block text-sm text-ink-gray-5 mb-1">
							{{ __('Batch Details') }}
						</label>
						<TextEditor
							:content="newBatch.batch_details"
							@change="(val) => (newBatch.batch_details = val)"
							:editable="true"
							:fixedMenu="true"
							:required="true"
							editorClass="prose-sm max-w-none border-b border-x bg-surface-gray-2 rounded-b-md py-1 px-2 min-h-[7rem] max-h-[20rem] overflow-y-scroll"
						/>
					</div>
				</div>
			</template>
		</Dialog>

		<Dialog v-model="showEventModal" :options="{ size: 'md' }">
			<template #body-content>
				<div v-if="selectedEvent" class="bg-white dark:bg-gray-900 rounded-lg overflow-hidden">

					<div class="h-2 w-full" :style="{ backgroundColor: selectedEvent.color }"></div>

					<div class="p-6">
						<div class="flex items-start justify-between gap-4 mb-6">
							<div>
								<h2 class="text-xl font-bold text-gray-900 dark:text-white leading-tight">
									{{ selectedEvent.title }}
								</h2>
								<div class="mt-2 inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-gray-100 dark:bg-gray-800 text-gray-800 dark:text-gray-300">
									{{ getTypeLabel(selectedEvent.type) }}
								</div>
							</div>
						</div>

						<div class="grid grid-cols-1 md:grid-cols-2 gap-y-4 gap-x-6 mb-6">
							<div class="flex items-start gap-3">
								<Calendar class="w-5 h-5 text-gray-400 dark:text-gray-500 mt-0.5" />
								<div>
									<div class="text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wide">
										{{ __('Date') }}
									</div>
									<div class="text-sm text-gray-900 dark:text-gray-100 font-medium mt-0.5">
										{{ dayjs(selectedEvent.date).format('dddd, D MMM YYYY') }}
									</div>
								</div>
							</div>

							<div class="flex items-start gap-3" v-if="selectedEvent.startTime !== '00:00'">
								<Clock class="w-5 h-5 text-gray-400 dark:text-gray-500 mt-0.5" />
								<div>
									<div class="text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wide">
										{{ __('Time') }}
									</div>
									<div class="text-sm text-gray-900 dark:text-gray-100 font-medium mt-0.5">
										{{ formatTime(selectedEvent.startTime) }} - {{ formatTime(selectedEvent.endTime) }}
									</div>
								</div>
							</div>

							<div class="flex items-start gap-3 md:col-span-2" v-if="selectedEvent.batchName || selectedEvent.courseName">
								<BookOpen class="w-5 h-5 text-gray-400 dark:text-gray-500 mt-0.5" />
								<div>
									<div class="text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wide">
										{{ selectedEvent.type === 'course' ? __('Course') : __('Batch') }}
									</div>
									<div class="text-sm text-gray-900 dark:text-gray-100 font-medium mt-0.5">
										{{ selectedEvent.batchName || selectedEvent.courseName }}
									</div>
								</div>
							</div>
						</div>

						<div v-if="selectedEvent.description" class="mb-6">
							<div class="flex items-start gap-3">
								<AlignLeft class="w-5 h-5 text-gray-400 dark:text-gray-500 mt-0.5" />
								<div class="w-full">
									<div class="text-xs font-medium text-gray-500 dark:text-gray-400 uppercase tracking-wide mb-1">
										{{ __('Description') }}
									</div>
									<div class="text-sm text-gray-700 dark:text-gray-300 bg-gray-50 dark:bg-gray-800 rounded p-3 leading-relaxed whitespace-pre-wrap">
										{{ selectedEvent.description }}
									</div>
								</div>
							</div>
						</div>

						<div class="flex justify-end gap-3 pt-4 border-t dark:border-gray-700">
							<a
								v-if="selectedEvent.type === 'liveClass' && selectedEvent.joinUrl"
								:href="selectedEvent.joinUrl"
								target="_blank"
								class="inline-flex items-center gap-2 px-4 py-2 bg-red-600 text-white text-sm font-medium rounded-md hover:bg-red-700 transition-colors shadow-sm"
							>
								<Video class="h-4 w-4" />
								{{ __('Join Class') }}
							</a>

							<router-link
								v-if="selectedEvent.type === 'batch'"
								:to="{
                  name: 'BatchDetail',
                  params: { batchName: selectedEvent.batchName }
                }"
								class="inline-flex items-center gap-2 px-4 py-2 bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600 text-gray-700 dark:text-gray-200 text-sm font-medium rounded-md hover:bg-gray-50 dark:hover:bg-gray-700 transition-colors"
							>
								{{ __('View Batch') }}
							</router-link>

							<router-link
								v-if="selectedEvent.type === 'course'"
								:to="{
                  name: 'CourseDetail',
                  params: { courseName: selectedEvent.courseName }
                }"
								class="inline-flex items-center gap-2 px-4 py-2 bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600 text-gray-700 dark:text-gray-200 text-sm font-medium rounded-md hover:bg-gray-50 dark:hover:bg-gray-700 transition-colors"
							>
								{{ __('View Course') }}
							</router-link>
						</div>
					</div>
				</div>
			</template>
		</Dialog>
	</div>
</template>

<script setup>
import { ref, computed, inject, onMounted, watch, reactive } from 'vue'
import { Button, Dialog, createResource, createListResource, FormControl, TextEditor, toast } from 'frappe-ui'
// Добавлены новые иконки для модального окна
import {
	ChevronLeft,
	ChevronRight,
	Video,
	Calendar,
	Clock,
	AlignLeft,
	BookOpen,
	Plus
} from 'lucide-vue-next'

const dayjs = inject('$dayjs')
const user = inject('$user')

const view = ref('month')
const currentDate = ref(dayjs().toDate())

const filters = ref({
	batches: true,
})

const eventColors = {
	batch: '#4285f4',
}

const weekDays = computed(() => {
	const days = []
	for (let i = 0; i < 7; i++) {
		days.push(dayjs().day(i).format('ddd'))
	}
	return days
})

const hours = computed(() => {
	const h = []
	for (let i = 0; i < 24; i++) {
		h.push(dayjs().hour(i).minute(0).format('h A'))
	}
	return h
})

const currentPeriodTitle = computed(() => {
	if (view.value === 'month') {
		return dayjs(currentDate.value).format('MMMM YYYY')
	} else if (view.value === 'week') {
		const start = weekDates.value[0]
		const end = weekDates.value[6]
		return `${dayjs(start).format('MMM D')} - ${dayjs(end).format('MMM D, YYYY')}`
	} else {
		return dayjs(currentDate.value).format('MMMM D, YYYY')
	}
})

const monthDates = computed(() => {
	const start = dayjs(currentDate.value).startOf('month').startOf('week')
	const end = dayjs(currentDate.value).endOf('month').endOf('week')
	const dates = []
	let date = start
	while (date.isBefore(end) || date.isSame(end, 'day')) {
		dates.push(date.toDate())
		date = date.add(1, 'day')
	}
	return dates
})

const weekDates = computed(() => {
	const start = dayjs(currentDate.value).startOf('week')
	const dates = []
	for (let i = 0; i < 7; i++) {
		dates.push(start.add(i, 'day').toDate())
	}
	return dates
})

const isAdmin = computed(() => {
	return (
		user.data?.is_moderator ||
		user.data?.is_instructor ||
		user.data?.is_evaluator
	)
})

const batches = createListResource({
	doctype: 'LMS Batch',
	url: 'lms.lms.utils.get_batches',
	filters: {},
	fields: ['name', 'title', 'start_date', 'end_date', 'start_time', 'end_time', 'timezone', 'category'],
	auto: true,
})

const myLiveClasses = createResource({
	url: 'lms.lms.api.get_my_live_classes',
	auto: false,
})

const adminLiveClasses = createResource({
	url: 'lms.lms.api.get_admin_live_classes',
	auto: false,
})

watch(
	() => [isAdmin.value, user.data],
	() => {
		if (user.data) {
			if (isAdmin.value) {
				adminLiveClasses.reload()
			} else {
				myLiveClasses.reload()
			}
		}
	},
	{ immediate: true }
)

const courses = createResource({
	url: 'lms.lms.api.get_my_courses',
	auto: true,
})

const allEvents = computed(() => {
	const events = []

	if (filters.value.batches && batches.data?.length) {
		batches.data.forEach((batch) => {
			if (batch.start_date && batch.end_date) {
				const startDate = dayjs(batch.start_date)
				const endDate = dayjs(batch.end_date)
				let date = startDate
				while (date.isBefore(endDate) || date.isSame(endDate, 'day')) {
					events.push({
						id: `batch-${batch.name}-${date.format('YYYY-MM-DD')}`,
						type: 'batch',
						title: batch.title || batch.name,
						date: date.format('YYYY-MM-DD'),
						startTime: batch.start_time || '00:00',
						endTime: batch.end_time || '23:59',
						color: eventColors.batch,
						textColor: '#ffffff',
						batchName: batch.name,
						dateTime: `${date.format('MMM D, YYYY')} ${formatTime(batch.start_time)} - ${formatTime(batch.end_time)}`,
					})
					date = date.add(1, 'day')
				}
			}
		})
	}

	// if (filters.value.liveClasses && liveClasses.value?.length) {
	// 	liveClasses.value.forEach((liveClass) => {
	// 		if (liveClass.date && liveClass.time) {
	// 			const startTime = dayjs(`${liveClass.date}T${liveClass.time}`)
	// 			const endTime = startTime.add(liveClass.duration || 60, 'minute')
	// 			events.push({
	// 				id: `liveclass-${liveClass.name}`,
	// 				type: 'liveClass',
	// 				title: liveClass.title,
	// 				description: liveClass.description,
	// 				date: liveClass.date,
	// 				startTime: liveClass.time,
	// 				endTime: endTime.format('HH:mm'),
	// 				color: eventColors.liveClass,
	// 				textColor: '#ffffff',
	// 				joinUrl: liveClass.join_url,
	// 				startUrl: liveClass.start_url,
	// 				batchName: liveClass.batch_name,
	// 				dateTime: `${dayjs(liveClass.date).format('MMM D, YYYY')} ${formatTime(liveClass.time)} - ${endTime.format('h:mm A')}`,
	// 			})
	// 		}
	// 	})
	// }
	//
	// if (filters.value.courses && courses.data?.length) {
	// 	courses.data.forEach((course) => {
	// 		const courseDate = course.enrollment_date || course.creation || dayjs().format('YYYY-MM-DD')
	// 		events.push({
	// 			id: `course-${course.name}`,
	// 			type: 'course',
	// 			title: course.title || course.name,
	// 			date: courseDate,
	// 			startTime: '00:00',
	// 			endTime: '23:59',
	// 			color: eventColors.course,
	// 			textColor: '#ffffff',
	// 			courseName: course.name,
	// 			dateTime: dayjs(courseDate).format('MMM D, YYYY'),
	// 		})
	// 	})
	// }
	return events
})

const isCurrentMonth = (date) => dayjs(date).isSame(currentDate.value, 'month')
const isToday = (date) => dayjs(date).isSame(dayjs(), 'day')
const getEventsForDate = (date) => {
	const dateStr = dayjs(date).format('YYYY-MM-DD')
	return allEvents.value.filter((event) => event.date === dateStr)
}
const getEventPosition = (event, date) => {
	const eventStart = dayjs(`${event.date}T${event.startTime}`)
	const dayStart = dayjs(date).startOf('day')
	return (eventStart.diff(dayStart, 'minute') / (24 * 60)) * 100
}
const getEventHeight = (event) => {
	const start = dayjs(`${event.date}T${event.startTime}`)
	const end = dayjs(`${event.date}T${event.endTime}`)
	return (end.diff(start, 'minute') / (24 * 60)) * 100
}

const formatTime = (timeString) => {
	if (!timeString) return ''
	const [hour, minute] = timeString.split(':').map(Number)
	const dummyDate = new Date(0, 0, 0, hour, minute)
	return new Intl.DateTimeFormat('en-US', {
		hour: 'numeric',
		minute: 'numeric',
		hour12: true,
	}).format(dummyDate)
}

const goToToday = () => { currentDate.value = dayjs().toDate() }

const previousPeriod = () => {
	if (view.value === 'month') currentDate.value = dayjs(currentDate.value).subtract(1, 'month').toDate()
	else if (view.value === 'week') currentDate.value = dayjs(currentDate.value).subtract(1, 'week').toDate()
	else currentDate.value = dayjs(currentDate.value).subtract(1, 'day').toDate()
}

const nextPeriod = () => {
	if (view.value === 'month') currentDate.value = dayjs(currentDate.value).add(1, 'month').toDate()
	else if (view.value === 'week') currentDate.value = dayjs(currentDate.value).add(1, 'week').toDate()
	else currentDate.value = dayjs(currentDate.value).add(1, 'day').toDate()
}

const showEventModal = ref(false)
const selectedEvent = ref(null)

const openEventModal = (event) => {
	selectedEvent.value = event
	showEventModal.value = true
}

const showCreateBatchModal = ref(false)
const newBatch = reactive({
	title: '',
	start_date: '',
	end_date: '',
	description: '',
	start_time: '',
	end_time: '',
	batch_details: '',
})

const createBatchResource = createResource({
	url: 'frappe.client.insert',
	makeParams() {
		return {
			doc: {
				doctype: 'LMS Batch',
				...newBatch,
			},
		}
	},
})

const createBatch = (close) => {
	if (!newBatch.title) {
		toast.error(__('Please enter a title'))
		return
	}
	if (!newBatch.start_date) {
		toast.error(__('Please select a start date'))
		return
	}
	if (!newBatch.end_date) {
		toast.error(__('Please select an end date'))
		return
	}
	if (!newBatch.start_time) {
		toast.error(__('Please select a start time'))
		return
	}
	if (!newBatch.end_time) {
		toast.error(__('Please select an end time'))
		return
	}

	createBatchResource.submit(
		{},
		{
			onSuccess(data) {
				toast.success(__('Batch created successfully'))
				batches.reload()
				resetBatchForm()
				close()
			},
			onError(err) {
				toast.error(err.messages?.[0] || err)
			},
		}
	)
}

const resetBatchForm = () => {
	newBatch.title = ''
	newBatch.start_date = ''
	newBatch.end_date = ''
	newBatch.description = ''
	newBatch.start_time = ''
	newBatch.end_time = ''
	newBatch.batch_details = ''
}

const getTypeLabel = (type) => {
	const map = {
		batch: 'Batch',
		liveClass: 'Live Class',
		course: 'Course'
	}
	return map[type] || type
}

watch(view, () => {
	if (view.value === 'week') {
		currentDate.value = dayjs(currentDate.value).startOf('week').toDate()
	}
})

watch(showCreateBatchModal, (newVal) => {
	if (!newVal) {
		// Сброс формы при закрытии модального окна
		resetBatchForm()
	}
})
</script>

<style scoped>
.calendar-schedule {
	padding: 1.5rem;
}

.calendar-container {
	min-height: 600px;
}

.month-view {
	min-height: 600px;
}

.week-view,
.day-view {
	min-height: 800px;
}

.line-clamp-2 {
	display: -webkit-box;
	-webkit-line-clamp: 2;
	-webkit-box-orient: vertical;
	overflow: hidden;
	line-clamp: 2;
}

/* Scrollbar styling for dark mode compatibility if needed */
::-webkit-scrollbar {
	width: 8px;
	height: 8px;
}
::-webkit-scrollbar-track {
	background: transparent;
}
::-webkit-scrollbar-thumb {
	background: #cbd5e1;
	border-radius: 4px;
}
.dark ::-webkit-scrollbar-thumb {
	background: #475569;
}
</style>
