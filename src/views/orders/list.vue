<template>
    <div class="container mt-4 orders-page" v-if="checkPermission(['order_view'])">

        <!-- ============ کارت آمار و Badge‌ها ============ -->
        <div class="row g-3 mb-4">
            <!-- کل سفارشات -->
            <div class="col-md-2">
                <div class="card bg-primary text-white">
                    <div class="card-body">
                        <div class="d-flex justify-content-between align-items-center">
                            <div>
                                <h6 class="mb-0">کل سفارشات</h6>
                                <h3 class="mb-0">{{ stats.total_orders }}</h3>
                            </div>
                            <i class="bi bi-cart fs-1"></i>
                        </div>
                    </div>
                </div>
            </div>

            <!-- پرداخت شده -->
            <div class="col-md-2">
                <div class="card bg-success text-white">
                    <div class="card-body">
                        <div class="d-flex justify-content-between align-items-center">
                            <div>
                                <h6 class="mb-0">پرداخت شده</h6>
                                <h3 class="mb-0">{{ stats.paid_orders }}</h3>
                            </div>
                            <i class="bi bi-check-circle fs-1"></i>
                        </div>
                    </div>
                </div>
            </div>

            <!-- لغو شده -->
            <div class="col-md-2">
                <div class="card bg-danger text-white">
                    <div class="card-body">
                        <div class="d-flex justify-content-between align-items-center">
                            <div>
                                <h6 class="mb-0">لغو شده</h6>
                                <h3 class="mb-0">{{ stats.cancelled_orders }}</h3>
                            </div>
                            <i class="bi bi-x-circle fs-1"></i>
                        </div>
                    </div>
                </div>
            </div>

            <!-- پرداخت از کیف پول -->
            <div class="col-md-2">
                <div class="card" style="background-color: #6f42c1; color: white;">
                    <div class="card-body">
                        <div class="d-flex justify-content-between align-items-center">
                            <div>
                                <h6 class="mb-0">کیف پول</h6>
                                <h3 class="mb-0">{{ stats.wallet_payments }}</h3>
                            </div>
                            <i class="bi bi-wallet2 fs-1"></i>
                        </div>
                    </div>
                </div>
            </div>

            <!-- پرداخت از درگاه -->
            <div class="col-md-2">
                <div class="card bg-warning text-dark">
                    <div class="card-body">
                        <div class="d-flex justify-content-between align-items-center">
                            <div>
                                <h6 class="mb-0">درگاه پرداخت</h6>
                                <h3 class="mb-0">{{ stats.online_payments }}</h3>
                            </div>
                            <i class="bi bi-credit-card fs-1"></i>
                        </div>
                    </div>
                </div>
            </div>

            <!-- جمع فروش -->
            <div class="col-md-2">
                <div class="card bg-info text-white">
                    <div class="card-body">
                        <div class="d-flex justify-content-between align-items-center">
                            <div>
                                <h6 class="mb-0">جمع فروش</h6>
                                <h5 class="mb-0">{{ formatPrice(stats.total_sales) }}</h5>
                            </div>
                            <i class="bi bi-coin fs-1"></i>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="card mb-3">
            <div class="card-header d-flex justify-content-between align-items-center">
                <h3>
                    <i class="bi bi-list-check"></i>
                    <span>مدیریت سفارش‌ها</span>
                </h3>
                <div class="d-flex align-items-center gap-2">
                    <select v-model="printType" class="form-select form-select-sm" style="width: 200px;">
                        <option value="full">پرینت کامل (جزئیات سفارش)</option>
                        <option value="label">پرینت برچسب (فرستنده و گیرنده)</option>
                    </select>

                    <button v-if="selectedOrders.length > 0" @click="goToPrint" class="btn btn-success btn-sm">
                        <i class="bi bi-printer"></i>
                        پرینت ({{ selectedOrders.length }})
                    </button>
                    <router-link to="/orders/create" class="btn btn-primary btn-sm">
                        <i class="bi bi-plus"></i>
                        افزودن سفارش
                    </router-link>
                </div>
            </div>

            <!-- فیلترهای جستجو -->
            <div class="card-body">
                <div class="row g-2">
                    <div class="col-md-3">
                        <label class="form-label">جستجوی هدفمند</label>
                        <input v-model="filters.search" type="text" class="form-control"
                            placeholder="جستجو (کاربر یا شماره سفارش)" />
                    </div>
                    <div class="col-md-3">
                        <label class="form-label">وضعیت سفارش</label>
                        <select v-model="filters.status" class="form-select">
                            <option value="">همه وضعیت‌ها</option>
                            <option value="pending">در انتظار</option>
                            <option value="paid">پرداخت شده</option>
                            <option value="shipped">ارسال شده</option>
                            <option value="completed">تکمیل شده</option>
                            <option value="failed">لغو شده</option>
                            <option value="returned">مرجوعی</option>
                        </select>
                    </div>
                    <div class="col-md-3">
                        <label class="form-label">وضعیت پرداخت</label>
                        <select v-model="filters.payment_status" class="form-select">
                            <option value="">همه پرداخت‌ها</option>
                            <option value="pending">در انتظار پرداخت</option>
                            <option value="paid">پرداخت شده</option>
                            <option value="failed">ناموفق</option>
                            <option value="refunded">برگشت داده شده</option>
                        </select>
                    </div>
                    <div class="col-md-3">
                        <label class="form-label">نحوه پرداخت</label>
                        <select v-model="filters.payment_method" class="form-select">
                            <option value="">همه روش‌ها</option>
                            <option value="online">پرداخت آنلاین</option>
                            <option value="wallet">کیف پول</option>
                            <option value="cod">پرداخت در محل</option>
                        </select>
                    </div>
                    <div class="col-md-3">
                        <label class="form-label">از تاریخ</label>
                        <date-picker display-format="jYYYY/jMM/jDD" placeholder="از تاریخ" format="YYYY-MM-DD"
                            v-model="filters.from_date"></date-picker>
                    </div>
                    <div class="col-md-4">
                        <label class="form-label">تا تاریخ</label>
                        <date-picker display-format="jYYYY/jMM/jDD" placeholder="تا تاریخ" format="YYYY-MM-DD"
                            v-model="filters.to_date"></date-picker>
                    </div>
                    <div class="col-md-1 d-flex align-items-end">
                        <button class="btn btn-primary w-100" @click="getOrders()" :disabled="loading">
                            <i class="bi bi-search"></i>
                        </button>
                    </div>
                    <div class="col-md-1 d-flex align-items-end">
                        <button class="btn btn-secondary w-100" @click="resetFilters()">
                            <i class="bi bi-arrow-counterclockwise"></i>
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Table -->
        <div class="card">
            <div class="card-body table-responsive">
                <table class="table table-bordered align-middle text-center">
                    <thead>
                        <tr>
                            <th style="width: 50px;">
                                <input type="checkbox" :checked="allSelected" @change="toggleAll"
                                    :disabled="orders.data.length === 0" />
                            </th>
                            <th>#</th>
                            <th>کاربر</th>
                            <th>آدرس</th>
                            <th>روش ارسال</th>
                            <th>مبلغ کل</th>
                            <th>وضعیت سفارش</th>
                            <th>وضعیت پرداخت</th>
                            <th>روش پرداخت</th>
                            <th>تاریخ</th>
                            <th style="width: 120px;">عملیات</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-if="loading">
                            <td colspan="10" class="text-center">
                                <div class="spinner-border" role="status"></div>
                            </td>
                        </tr>
                        <tr v-else v-for="order in orders.data" :key="order.id">
                            <td>
                                <input type="checkbox" :value="order.id" v-model="selectedOrders" />
                            </td>
                            <td>{{ order.id }}</td>
                            <td>{{ order.user?.full_name ?? "-" }}</td>
                            <td>{{ order.address?.address_line ?? "-" }}</td>
                            <td>{{ order.shipping?.title ?? "-" }}</td>
                            <td>{{ Number(order.total).toLocaleString('fa-Ir') }} تومان</td>
                            <td>
                                <span class="badge" :class="statusBadge(order.status)">
                                    {{ statusText(order.status) }}
                                </span>
                            </td>
                            <td>
                                <span class="badge" :class="paymentStatusBadge(order.payment_status)">
                                    {{ paymentStatusText(order.payment_status) }}
                                </span>
                            </td>
                            <td>{{ paymentMethodText(order.payment_method) }}</td>
                            <td>
                                {{ new Date(order.created_at).toLocaleDateString('fa') }}
                            </td>
                            <td>
                                <router-link :to="`/orders/${order.id}`" class="btn btn-sm btn-info">
                                    <i class="bi bi-eye"></i>
                                </router-link>
                                <button @click="singlePrint(order.id)" class="btn btn-sm btn-secondary">
                                    <i class="bi bi-printer"></i>
                                </button>
                            </td>
                        </tr>
                        <tr v-if="!loading && orders.data.length === 0">
                            <td colspan="10" class="text-center">هیچ سفارشی یافت نشد</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

        <b-pagination v-model="orders.current_page" :total-rows="orders.total" v-if="orders.last_page != 1"
            :per-page="orders.per_page" @Update:modelValue="changePage" align="center" class="mt-3"></b-pagination>

    </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";
import { useAdmin } from '@/stores/modules/admin';

const router = useRouter();
const store = useAdmin();
const checkPermission = store.checkPermission;

const orders = ref({ data: [] });
const loading = ref(false);
const selectedOrders = ref([]);
const printType = ref('full');

const stats = ref({
    total_orders: 0,
    paid_orders: 0,
    cancelled_orders: 0,
    wallet_payments: 0,
    online_payments: 0,
    total_sales: 0,
    total_discount: 0,
});

const filters = ref({
    search: "",
    status: "",
    payment_status: "",
    from_date: "",
    to_date: "",
    payment_method: "",
});
const currentPage = ref(1);

const allSelected = computed(() => {
    return orders.value.data.length > 0 &&
        orders.value.data.every(order => selectedOrders.value.includes(order.id));
});

const toggleAll = (event) => {
    if (event.target.checked) {
        selectedOrders.value = orders.value.data.map(order => order.id);
    } else {
        selectedOrders.value = [];
    }
};

const resetFilters = () => {
    filters.value = {
        search: "",
        status: "",
        payment_status: "",
        from_date: "",
        to_date: "",
        payment_method: "",
    };
    getOrders(1);
};

const formatPrice = (price) => {
    if (!price) return '0 تومان';
    return Number(price).toLocaleString('fa-Ir') + ' تومان';
};

const goToPrint = () => {
    if (selectedOrders.value.length === 0) {
        alert('لطفاً حداقل یک سفارش را انتخاب کنید.');
        return;
    }
    router.push({
        path: '/orders/print',
        query: {
            ids: selectedOrders.value.join(','),
            type: printType.value
        }
    });
};

const singlePrint = (orderId) => {
    router.push({
        path: '/orders/print',
        query: {
            ids: orderId.toString(),
            type: printType.value
        }
    });
};

const getOrders = async (page = 1) => {
    loading.value = true;
    selectedOrders.value = [];
    try {
        const response = await axios.get("/orders", {
            params: {
                page,
                ...filters.value,
            },
        });
        orders.value = response.data.data;
        if (response.data.stats) {
            stats.value = response.data.stats;
        }
        currentPage.value = page;
    } catch (error) {
        console.error('Error fetching orders:', error);
    } finally {
        loading.value = false;
    }
};

const changePage = (page) => {
    if (page) getOrders(page);
};

// helpers
const statusText = (status) => {
    const map = {
        pending: "در انتظار",
        failed: "ناموفق",
        card_transfer_pending: "در انتظار کارت به کارت",
        
        processing: "در حال پردازش",
        paid: "پرداخت شده",
        shipped: "ارسال شده",
        completed: "تکمیل شده",
        canceled: "لغو شده",
        cancelled: "لغو شده",
        returned: "مرجوعی",
    };
    return map[status] ?? status;
};

const statusBadge = (status) => {
    const map = {
        pending: "bg-secondary",
        failed: "bg-warning text-dark",
        processing: "bg-info",
        shipped: "bg-primary",
        completed: "bg-success",
        paid: "bg-success",
        canceled: "bg-danger",
        cancelled: "bg-danger",
        returned: "bg-dark",
    };
    return map[status] ?? "bg-secondary";
};

const paymentStatusText = (status) => {
    const map = {
        pending: "در انتظار پرداخت",
        paid: "پرداخت شده",
        failed: "ناموفق",
        refunded: "برگشت داده شده",
    };
    return map[status] ?? status;
};

const paymentStatusBadge = (status) => {
    const map = {
        pending: "bg-warning text-dark",
        paid: "bg-success",
        failed: "bg-danger",
        refunded: "bg-secondary",
    };
    return map[status] ?? "bg-secondary";
};

const paymentMethodText = (method) => {
    const map = {
        online: "پرداخت آنلاین",
        wallet: "کیف پول",
        cod: "پرداخت در محل",
       card_transfer : "کارت به کارت",
    };
    return map[method] ?? method;
};

onMounted(() => {
    getOrders(1);
});
</script>