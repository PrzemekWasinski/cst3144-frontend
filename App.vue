<script setup>
import { ref, computed } from 'vue';

const sitename = ref('After School Lessons');
const lessons = ref([
    { _id: 1, subject: 'Math', location: 'London', price: 100, spaces: 5, icon: 'fas fa-calculator' },
    { _id: 2, subject: 'English', location: 'Oxford', price: 80, spaces: 5, icon: 'fas fa-book' },
    { _id: 3, subject: 'Science', location: 'London', price: 90, spaces: 5, icon: 'fas fa-flask' },
    { _id: 4, subject: 'Music', location: 'Bristol', price: 70, spaces: 5, icon: 'fas fa-music' },
    { _id: 5, subject: 'Art', location: 'London', price: 85, spaces: 5, icon: 'fas fa-palette' },
    { _id: 6, subject: 'History', location: 'Cambridge', price: 75, spaces: 5, icon: 'fas fa-landmark' },
    { _id: 7, subject: 'Geography', location: 'Manchester', price: 65, spaces: 5, icon: 'fas fa-globe' },
    { _id: 8, subject: 'PE', location: 'London', price: 60, spaces: 5, icon: 'fas fa-running' },
    { _id: 9, subject: 'Drama', location: 'Oxford', price: 95, spaces: 5, icon: 'fas fa-theater-masks' },
    { _id: 10, subject: 'Computer Science', location: 'London', price: 110, spaces: 5, icon: 'fas fa-laptop-code' }
]);

const sortAttribute = ref('subject');
const sortOrder = ref('asc');

const sortedLessons = computed(() => {
    let tempLessons = [...lessons.value];

    return tempLessons.sort((a, b) => {
        let aValue = a[sortAttribute.value];
        let bValue = b[sortAttribute.value];

        if (sortAttribute.value === 'price' || sortAttribute.value === 'spaces') {
            aValue = Number(aValue);
            bValue = Number(bValue);
        } else {
            aValue = String(aValue).toLowerCase();
            bValue = String(bValue).toLowerCase();
        }

        if (aValue < bValue) {
            return sortOrder.value === 'asc' ? -1 : 1;
        }
        if (aValue > bValue) {
            return sortOrder.value === 'asc' ? 1 : -1;
        }
        return 0;
    });
});
</script>

<template>
    <div id="app-container" class="container mt-5">
        <header class="mb-4">
            <h1>{{ sitename }}</h1>
        </header>

        <main>
            <!-- Sort Controls -->
            <div class="row mb-4">
                <div class="col-md-12">
                    <div class="d-flex justify-content-end">
                        <div class="me-2">
                            <label for="sort-attribute" class="form-label">Sort by:</label>
                            <select id="sort-attribute" class="form-select" v-model="sortAttribute">
                                <option value="subject">Subject</option>
                                <option value="location">Location</option>
                                <option value="price">Price</option>
                                <option value="spaces">Availability</option>
                            </select>
                        </div>
                        <div>
                            <label for="sort-order" class="form-label">Order:</label>
                            <div class="btn-group">
                                <input type="radio" class="btn-check" name="sort-order" id="asc" value="asc"
                                    v-model="sortOrder" autocomplete="off" checked>
                                <label class="btn btn-outline-secondary" for="asc">
                                    <i class="fas fa-arrow-up"></i> Asc
                                </label>
                                <input type="radio" class="btn-check" name="sort-order" id="desc" value="desc"
                                    v-model="sortOrder" autocomplete="off">
                                <label class="btn btn-outline-secondary" for="desc">
                                    <i class="fas fa-arrow-down"></i> Desc
                                </label>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Lessons Grid -->
            <div class="row">
                <div v-for="lesson in sortedLessons" :key="lesson._id" class="col-md-4 mb-4">
                    <div class="card h-100">
                        <div class="card-body">
                            <h5 class="card-title"><i :class="lesson.icon"></i> {{ lesson.subject }}</h5>
                            <p class="card-text"><strong>Location:</strong> {{ lesson.location }}</p>
                            <p class="card-text"><strong>Price:</strong> £{{ lesson.price }}</p>
                            <p class="card-text"><strong>Spaces:</strong> {{ lesson.spaces }}</p>
                        </div>
                    </div>
                </div>
            </div>
        </main>
    </div>
</template>

<style>
.card {
    transition: transform 0.2s;
}

.card:hover {
    transform: translateY(-5px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.card-title i {
    margin-right: 10px;
    color: #0d6efd;
}

.btn-check:checked+.btn {
    background-color: #0d6efd;
    color: white;
}
</style>
