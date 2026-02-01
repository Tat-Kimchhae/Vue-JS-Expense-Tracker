<template>
    <Header />
    <div class="container">
        <Balance :balance="+balance" />
        <IncomeExpenses :income="+income" :expense="+expense" />
        <TransactionList :transactions="transactions" @handleDeletedTransaction="handleDeletedTransaction" />
        <AddTransaction @transactionSubmitted="handleTransactionSubmitted" />
    </div>
</template>

<script setup>
import Header from './components/Header.vue';
import Balance from './components/Balance.vue';
import IncomeExpenses from './components/IncomeExpenses.vue';
import TransactionList from './components/TransactionList.vue';
import AddTransaction from './components/AddTransaction.vue';
import { computed, ref, onMounted } from 'vue';
import { useToast } from 'vue-toastification';

const transactions = ref([]);

onMounted(() => {
    const savedTransactions = JSON.parse(localStorage.getItem('transactions'));
    if (savedTransactions) {
        transactions.value = savedTransactions;
    }
});

// Get total
const balance = computed(() => {
    return transactions.value.reduce((accumulator, transaction) => {
        return accumulator + transaction.amount;
    }, 0).toFixed(2);
});

// Get income
const income = computed(() => {
    return transactions.value.filter((transaction) => {
        return transaction.amount > 0;
    }).reduce((accumulator, transaction) => {
        return accumulator + transaction.amount;
    }, 0).toFixed(2);
});

// Get expense
const expense = computed(() => {
    return transactions.value.filter((transaction) => {
        return transaction.amount < 0;
    }).reduce((accumulator, transaction) => {
        return accumulator + transaction.amount;
    }, 0).toFixed(2);
});

const toast = useToast();

// Handle submit
const handleTransactionSubmitted = (transactionData) => {
    transactions.value.push({
        id: generateUnqiueID(),
        text: transactionData.text,
        amount: transactionData.amount
    });
    
    saveToLocalStorage();

    toast.success('Transaction added');
};

// Generate unique ID
const generateUnqiueID = () => {
    return Math.floor(Math.random() * 1000000);
}

// Handle deleted transaction
const handleDeletedTransaction = (id) => {
    transactions.value = transactions.value.filter((transaction) => {
        return transaction.id !== id;
    });

    saveToLocalStorage();
    
    toast.success('Transaction deleted');
}

// Save to local storage
const saveToLocalStorage = () => {
    localStorage.setItem('transactions', JSON.stringify(transactions.value));
};

</script>