<script setup>
import { ref, computed, reactive, onMounted } from 'vue';

// --- CONFIG ---
const API_BASE_URL = 'https://cst3144-backend-siah.onrender.com';

// --- STATE (REFS) ---
const sitename = ref('After School Lessons');
const lessons = ref([]);
const cart = ref([]);
const showLessons = ref(true);
const searchQuery = ref('');
const sortAttribute = ref('subject');
const sortOrder = ref('asc');
const isLoading = ref(true);
const searchTimeout = ref(null);

const order = reactive({
    name: '',
    phone: ''
});

const validations = reactive({
    isNameValid: true,
    isPhoneValid: true
});

// --- COMPUTED PROPERTIES ---
const cartItemCount = computed(() => {
    return cart.value.length;
});

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

const isFormValid = computed(() => {
    return order.name && order.phone && validations.isNameValid && validations.isPhoneValid;
});

// --- METHODS ---
function goHome() {
    showLessons.value = true;
    searchQuery.value = '';
    fetchLessons();
}

function toggleShowCart() {
    showLessons.value = !showLessons.value;
}

function addToCart(lesson) {
    if (lesson.spaces > 0) {
        lesson.spaces--;
        cart.value.push({ ...lesson });
    }
}

function removeFromCart(itemToRemove) {
    const index = cart.value.findIndex(item => item._id === itemToRemove._id);
    if (index !== -1) {
        cart.value.splice(index, 1);

        const originalLesson = lessons.value.find(lesson => lesson._id === itemToRemove._id);
        if (originalLesson) {
            originalLesson.spaces++;
        }
    }
}

function validateForm() {
    const nameRegex = /^[A-Za-z\s]+$/;
    const phoneRegex = /^[0-9]+$/;

    validations.isNameValid = order.name ? nameRegex.test(order.name) : true;
    validations.isPhoneValid = order.phone ? phoneRegex.test(order.phone) : true;
}

async function fetchLessons(query = '') {
    isLoading.value = true;
    try {
        const url = query ? `${API_BASE_URL}/search?q=${query}` : `${API_BASE_URL}/lessons`;
        const response = await fetch(url);
        if (!response.ok) throw new Error('Failed to fetch lessons');
        lessons.value = await response.json();
    } catch (error) {
        console.error('Fetch lessons error:', error);
        alert('Could not load lessons. Please check the backend connection.');
    } finally {
        isLoading.value = false;
    }
}

function performSearch() {
    clearTimeout(searchTimeout.value);
    searchTimeout.value = setTimeout(() => {
        fetchLessons(searchQuery.value);
    }, 300);
}

async function updateLessonSpaces(lessonId, newSpaces) {
    try {
        const response = await fetch(`${API_BASE_URL}/lessons/${lessonId}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ spaces: newSpaces })
        });
        if (!response.ok) throw new Error(`Failed to update spaces for lesson ${lessonId}`);
        return await response.json();
    } catch (error) {
        console.error('Update spaces error:', error);
    }
}

async function submitOrder() {
    if (!isFormValid.value) {
        alert('Please fill in the form correctly.');
        return;
    }

    const lessonIDs = cart.value.map(item => item._id);
    const spaces = cart.value.map(item => 1);

    const orderData = {
        name: order.name,
        phone: order.phone,
        lessonIDs: lessonIDs,
        spaces: spaces
    };

    try {
        // 1. Create the order
        const orderResponse = await fetch(`${API_BASE_URL}/orders`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(orderData)
        });

        if (!orderResponse.ok) throw new Error('Failed to create order');

        // 2. Update lesson spaces
        const updatePromises = cart.value.map(item => {
            return updateLessonSpaces(item._id, item.spaces);
        });
        await Promise.all(updatePromises);

        // 3. Show success and reset
        alert('Order Submitted Successfully!');
        cart.value = [];
        order.name = '';
        order.phone = '';
        showLessons.value = true;

    } catch (error) {
        console.error('Order submission error:', error);
        alert('There was an error submitting your order. Please try again.');
    }
}

// --- LIFECYCLE HOOK ---
onMounted(() => {
    fetchLessons();
});
</script>

<template>
    <div id="app-container" class="container mt-5">
        <header class="d-flex justify-content-between align-items-center mb-4">
            <h1 @click="goHome" style="cursor: pointer;">{{ sitename }}</h1>
            <button class="btn btn-primary" @click="toggleShowCart" :disabled="cart.length === 0">
                <i class="fas fa-shopping-cart"></i> Cart ({{ cartItemCount }})
            </button>
        </header>

        <main>
            <!-- Lessons Page -->
            <div v-if="showLessons">
                <!-- Search and Sort Controls -->
                <div class="row mb-4">
                    <div class="col-md-6">
                        <div class="input-group">
                            <span class="input-group-text"><i class="fas fa-search"></i></span>
                            <input type="text" class="form-control" placeholder="Search lessons..." v-model="searchQuery"
                                @input="performSearch">
                        </div>
                    </div>
                    <div class="col-md-6">
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
                <div v-if="isLoading" class="text-center">
                    <div class="spinner-border" role="status">
                        <span class="visually-hidden">Loading...</span>
                    </div>
                    <p>Loading lessons...</p>
                </div>
                <div v-else class="row">
                    <div v-for="lesson in sortedLessons" :key="lesson._id" class="col-md-4 mb-4">
                        <div class="card h-100">
                            <div class="card-body d-flex flex-column">
                                <h5 class="card-title"><i :class="lesson.icon"></i> {{ lesson.subject }}</h5>
                                <p class="card-text"><strong>Location:</strong> {{ lesson.location }}</p>
                                <p class="card-text"><strong>Price:</strong> £{{ lesson.price }}</p>
                                <p class="card-text"><strong>Spaces:</strong> {{ lesson.spaces }}</p>
                                <button class="btn btn-success mt-auto" @click="addToCart(lesson)"
                                    :disabled="lesson.spaces === 0">
                                    {{ lesson.spaces > 0 ? 'Add to Cart' : 'Sold Out' }}
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Shopping Cart Page -->
            <div v-else>
                <h2>Shopping Cart</h2>
                <div v-if="cart.length === 0" class="alert alert-info">
                    Your cart is empty.
                </div>
                <div v-else>
                    <div class="row">
                        <div v-for="(item, index) in cart" :key="index" class="col-md-6 mb-3">
                            <div class="card">
                                <div class="card-body">
                                    <h5 class="card-title"><i :class="item.icon"></i> {{ item.subject }}</h5>
                                    <p class="card-text"><strong>Location:</strong> {{ item.location }}</p>
                                    <p class="card-text"><strong>Price:</strong> £{{ item.price }}</p>
                                    <button class="btn btn-danger" @click="removeFromCart(item)">Remove</button>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- Checkout Form -->
                    <div class="mt-4">
                        <h3>Checkout</h3>
                        <form @submit.prevent="submitOrder">
                            <div class="mb-3">
                                <label for="name" class="form-label">Name</label>
                                <input type="text" id="name" class="form-control" v-model.trim="order.name"
                                    @input="validateForm" required>
                                <div v-if="!validations.isNameValid" class="text-danger">Name must contain letters only.
                                </div>
                            </div>
                            <div class="mb-3">
                                <label for="phone" class="form-label">Phone Number</label>
                                <input type="tel" id="phone" class="form-control" v-model.trim="order.phone"
                                    @input="validateForm" required>
                                <div v-if="!validations.isPhoneValid" class="text-danger">Phone must contain numbers
                                    only.</div>
                            </div>
                            <button type="submit" class="btn btn-primary" :disabled="!isFormValid">Place Order</button>
                        </form>
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

button:disabled {
    cursor: not-allowed;
}

.form-control.is-invalid {
    border-color: #dc3545;
}

.form-control.is-valid {
    border-color: #198754;
}
</style>
