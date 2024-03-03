<template>
  <Header />
  <div class="container">
    <!-- add sign for casting to number -->
    <Balance v-bind:total="+total"/>
    <IncomeExpenses :income="+income" :expenses="+expenses"/> 

    <!-- listening to custom event -->
    <TransactionList :transactions="transactions" @transactionDeleted="handleTransactionDeleted"/>
    <AddTransaction @transactionSubmitted="handleTransactionSubmitted"/>
  </div>
</template>

<script setup>
  import Header from './components/Header.vue';
  import Balance from './components/Balance.vue';
  import IncomeExpenses from './components/IncomeExpenses.vue';
  import TransactionList from './components/TransactionList.vue';
  import AddTransaction from './components/AddTransaction.vue';

  import { useToast } from 'vue-toastification';
  import { ref, computed, onMounted } from 'vue';

  const toast = useToast();

  const transactions = ref([]);

  onMounted(() => {
    const savedTransactions = JSON.parse(localStorage.getItem('transactions'));

    if (savedTransactions) {
      transactions.value = savedTransactions;
    }
  });

  // get total balance
  const total = computed(() => {
    return transactions.value.reduce((acc, transaction) => { return acc + transaction.amount; }, 0);
  });

  // get income
  const income = computed(() => {
    return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, transaction) => { return acc + transaction.amount; }, 0)
    .toFixed(2);
  });

  // get expenses
  const expenses = computed(() => {
    return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, transaction) => { return acc + transaction.amount; }, 0)
    .toFixed(2);
  });

  // a helper function to generate unique ID
  const generateUniqueId = () => {
    return Math.floor(Math.random() * 1000000);
  };

  // add transaction
  const handleTransactionSubmitted = (transactionData) => {
    transactions.value.push({
      id: generateUniqueId(),
      text: transactionData.text,
      amount: transactionData.amount,
    });

    saveTransactionToLocalStorage();

    toast.success('Transaction added');
  };

  // delete transaction
  const handleTransactionDeleted = (id) => {
    transactions.value = transactions.value.filter(
      (transaction) => transaction.id !== id
    );

    saveTransactionToLocalStorage();

    toast.success('Transaction deleted');
  };

  // save to local storage
  const saveTransactionToLocalStorage = () => {
    localStorage.setItem('transactions', JSON.stringify(transactions.value))
  }
  
</script>
