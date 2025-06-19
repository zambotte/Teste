<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestor Financeiro Pessoal</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f2f5;
            display: flex;
            justify-content: center;
            align-items: flex-start; /* Align items to the start for better layout */
            min-height: 100vh;
            padding: 20px;
            box-sizing: border-box;
        }
        .container {
            background-color: #ffffff;
            border-radius: 1rem; /* Rounded corners */
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
            width: 100%;
            max-width: 600px;
            padding: 1.5rem;
            box-sizing: border-box;
        }
        .input-group label {
            display: block;
            margin-bottom: 0.5rem;
            font-weight: 600;
            color: #374151;
        }
        .input-group input,
        .input-group select {
            width: 100%;
            padding: 0.75rem;
            border: 1px solid #d1d5db;
            border-radius: 0.5rem;
            font-size: 1rem;
            color: #374151;
            box-sizing: border-box;
            background-color: #f9fafb;
        }
        .btn {
            padding: 0.75rem 1.5rem;
            border-radius: 0.5rem;
            font-weight: 600;
            cursor: pointer;
            transition: background-color 0.2s ease-in-out, transform 0.1s ease-in-out;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        }
        .btn-primary {
            background-color: #4f46e5;
            color: #ffffff;
        }
        .btn-primary:hover {
            background-color: #4338ca;
            transform: translateY(-1px);
        }
        .btn-delete {
            background-color: #ef4444;
            color: #ffffff;
            padding: 0.4rem 0.8rem;
            font-size: 0.875rem;
            border-radius: 0.375rem;
            box-shadow: none;
        }
        .btn-delete:hover {
            background-color: #dc2626;
            transform: translateY(-1px);
        }
        .balance-card {
            background-color: #e0f2f7;
            border-radius: 0.75rem;
            padding: 1rem;
            text-align: center;
            margin-bottom: 1.5rem;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
        }
        .balance-card h2 {
            font-size: 1.125rem;
            font-weight: 600;
            color: #065f46;
            margin-bottom: 0.5rem;
        }
        .balance-card p {
            font-size: 2rem;
            font-weight: 700;
            color: #047857;
        }
        .transaction-list {
            max-height: 400px;
            overflow-y: auto;
            border-top: 1px solid #e5e7eb;
            padding-top: 1rem;
        }
        .transaction-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 0.75rem 0;
            border-bottom: 1px solid #e5e7eb;
        }
        .transaction-item:last-child {
            border-bottom: none;
        }
        .transaction-item .description {
            font-weight: 500;
            color: #1f2937;
        }
        .transaction-item .amount {
            font-weight: 700;
        }
        .transaction-item .amount.income {
            color: #059669; /* Green for income */
        }
        .transaction-item .amount.expense {
            color: #ef4444; /* Red for expense */
        }
        .message-box {
            position: fixed;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            background-color: #333;
            color: white;
            padding: 10px 20px;
            border-radius: 5px;
            z-index: 1000;
            opacity: 0;
            transition: opacity 0.5s ease-in-out;
        }
        .message-box.show {
            opacity: 1;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1 class="text-3xl font-bold text-center text-gray-800 mb-6">Meu Gestor Financeiro</h1>

        <div class="balance-card">
            <h2>Saldo Atual</h2>
            <p id="current-balance">R$ 0,00</p>
        </div>

        <form id="transaction-form" class="space-y-4 mb-8">
            <div class="input-group">
                <label for="description">Descrição</label>
                <input type="text" id="description" placeholder="Ex: Salário, Aluguel, Compras" required class="focus:ring-2 focus:ring-indigo-500 focus:border-transparent">
            </div>

            <div class="input-group">
                <label for="amount">Valor (R$)</label>
                <input type="number" id="amount" step="0.01" placeholder="Ex: 100.00" required class="focus:ring-2 focus:ring-indigo-500 focus:border-transparent">
            </div>

            <div class="input-group">
                <label for="type">Tipo</label>
                <select id="type" class="focus:ring-2 focus:ring-indigo-500 focus:border-transparent">
                    <option value="income">Receita</option>
                    <option value="expense">Despesa</option>
                </select>
            </div>

            <button type="submit" class="btn btn-primary w-full">Adicionar Transação</button>
        </form>

        <div>
            <h2 class="text-xl font-bold text-gray-700 mb-4">Histórico de Transações</h2>
            <div id="transaction-list" class="transaction-list">
                <!-- Transactions will be loaded here -->
            </div>
            <p id="no-transactions-message" class="text-gray-500 text-center mt-4 hidden">Nenhuma transação ainda. Adicione uma acima!</p>
        </div>
    </div>

    <div id="message-box" class="message-box"></div>

    <script type="module">
        // Firebase imports - These will be used if data persistence is added in the future.
        // For now, data is stored in localStorage.
        // import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        // import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        // import { getFirestore, doc, getDoc, addDoc, setDoc, updateDoc, deleteDoc, onSnapshot, collection, query, where, addDoc, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const transactionForm = document.getElementById('transaction-form');
        const descriptionInput = document.getElementById('description');
        const amountInput = document.getElementById('amount');
        const typeInput = document.getElementById('type');
        const currentBalanceDisplay = document.getElementById('current-balance');
        const transactionListContainer = document.getElementById('transaction-list');
        const noTransactionsMessage = document.getElementById('no-transactions-message');
        const messageBox = document.getElementById('message-box');

        let transactions = JSON.parse(localStorage.getItem('transactions')) || [];

        // Function to show a temporary message
        function showMessage(message) {
            messageBox.textContent = message;
            messageBox.classList.add('show');
            setTimeout(() => {
                messageBox.classList.remove('show');
            }, 3000);
        }

        // Function to update the balance and transaction list
        function updateUI() {
            let totalBalance = 0;
            transactionListContainer.innerHTML = ''; // Clear existing transactions

            if (transactions.length === 0) {
                noTransactionsMessage.classList.remove('hidden');
            } else {
                noTransactionsMessage.classList.add('hidden');
                transactions.forEach(transaction => {
                    const listItem = document.createElement('div');
                    listItem.classList.add('transaction-item');

                    const amountClass = transaction.type === 'income' ? 'income' : 'expense';
                    const sign = transaction.type === 'income' ? '+' : '-';
                    const formattedAmount = `${sign} R$ ${Math.abs(transaction.amount).toFixed(2).replace('.', ',')}`;

                    listItem.innerHTML = `
                        <span class="description">${transaction.description}</span>
                        <span class="amount ${amountClass}">${formattedAmount}</span>
                        <button class="btn btn-delete" data-id="${transaction.id}">Excluir</button>
                    `;
                    transactionListContainer.appendChild(listItem);

                    if (transaction.type === 'income') {
                        totalBalance += transaction.amount;
                    } else {
                        totalBalance -= transaction.amount;
                    }
                });
            }

            currentBalanceDisplay.textContent = `R$ ${totalBalance.toFixed(2).replace('.', ',')}`;
            localStorage.setItem('transactions', JSON.stringify(transactions));
        }

        // Handle form submission
        transactionForm.addEventListener('submit', (e) => {
            e.preventDefault();

            const description = descriptionInput.value.trim();
            const amount = parseFloat(amountInput.value);
            const type = typeInput.value;

            if (description === '' || isNaN(amount) || amount <= 0) {
                showMessage('Por favor, preencha todos os campos corretamente.');
                return;
            }

            const newTransaction = {
                id: Date.now(), // Unique ID for the transaction
                description,
                amount,
                type
            };

            transactions.push(newTransaction);
            updateUI(); // Update the display
            showMessage('Transação adicionada com sucesso!');

            // Clear the form
            descriptionInput.value = '';
            amountInput.value = '';
            typeInput.value = 'income'; // Reset to default
        });

        // Handle deleting a transaction
        transactionListContainer.addEventListener('click', (e) => {
            if (e.target.classList.contains('btn-delete')) {
                const transactionId = parseInt(e.target.dataset.id);
                transactions = transactions.filter(transaction => transaction.id !== transactionId);
                updateUI();
                showMessage('Transação excluída!');
            }
        });

        // Initial UI update when the page loads
        updateUI();
    </script>
</body>
</html>

