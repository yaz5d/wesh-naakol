<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>وش نأكل؟ - تصويت المطاعم الجماعي</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: "Cairo", sans-serif;
            background: linear-gradient(to top right, #f0f9ff, #e0f2fe); /* Light blue gradient */
        }
        /* Custom scrollbar for restaurant list */
        .scrollable-list::-webkit-scrollbar {
            width: 8px;
        }
        .scrollable-list::-webkit-scrollbar-track {
            background: #e0e7ff;
            border-radius: 10px;
        }
        .scrollable-list::-webkit-scrollbar-thumb {
            background: #a5b4fc;
            border-radius: 10px;
        }
        .scrollable-list::-webkit-scrollbar-thumb:hover {
            background: #818cf8;
        }
        /* Smooth transition for vote buttons */
        .vote-btn {
            transition: all 0.2s ease-in-out;
        }
        .vote-btn:active {
            transform: scale(0.95);
        }
        .action-btn {
             transition: all 0.2s ease-in-out;
             box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }
        .action-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 12px rgba(0,0,0,0.1);
        }
        /* Custom Dialog styles */
        .custom-dialog-backdrop {
            position: fixed;
            inset: 0;
            background-color: rgba(0, 0, 0, 0.6);
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 1rem;
            z-index: 1000;
        }
        .custom-dialog-content {
            background-color: white;
            padding: 1.5rem;
            border-radius: 0.75rem;
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
            max-width: 24rem;
            width: 100%;
            text-align: center;
            animation: fadeIn 0.3s ease-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: scale(0.95); }
            to { opacity: 1; transform: scale(1); }
        }
    </style>
</head>
<body class="min-h-screen flex flex-col items-center justify-center py-6 px-4 sm:px-6 lg:px-8">
    <div class="max-w-md w-full bg-white/70 backdrop-blur-xl p-8 rounded-2xl shadow-2xl shadow-blue-200/50 space-y-6">
        <h1 class="text-4xl font-black text-slate-800 text-center">
            وش نأكل؟
        </h1>
        <p class="mt-2 text-center text-sm text-slate-600">
            ساعد مجموعتك في اختيار المطعم المثالي عن طريق التصويت!
        </p>

        <div id="session-setup" class="space-y-4">
            <h2 class="text-xl font-bold text-slate-700 border-b-2 border-slate-200 pb-2">1. انضم للجلسة</h2>
            <div class="flex flex-col sm:flex-row gap-3">
                <input
                    type="text"
                    id="user-name-input"
                    placeholder="أدخل اسمك"
                    class="flex-grow p-3 border border-slate-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-cyan-500"
                    aria-label="اسمك"
                />
                <button
                    id="join-session-btn"
                    class="action-btn w-full sm:w-auto px-6 py-3 bg-cyan-600 text-white font-bold rounded-lg hover:bg-cyan-700 focus:outline-none focus:ring-2 focus:ring-cyan-500 focus:ring-offset-2"
                >
                    انضم
                </button>
            </div>
            <div id="joined-users" class="mt-4 p-3 bg-cyan-50 border border-cyan-200 rounded-lg">
                <h3 class="font-bold text-cyan-800">المشاركون:</h3>
                <ul id="users-list" class="list-disc list-inside text-cyan-700">
                    <li id="no-users-message" class="text-sm italic text-slate-500">لم ينضم أحد بعد.</li>
                </ul>
            </div>
        </div>

        <div id="restaurant-section" class="space-y-4 mt-6 hidden">
            <h2 class="text-xl font-bold text-slate-700 border-b-2 border-slate-200 pb-2">2. أضف مطاعم</h2>
            <div class="flex flex-col gap-3">
                <textarea
                    id="bulk-restaurant-input"
                    rows="4"
                    placeholder="أدخل قائمة مطاعم (كل مطعم في سطر جديد، أو مفصولة بفاصلة)&#10;مثال:&#10;شاورما كلاسك&#10;برجر فيول، بيتزا هت"
                    class="p-3 border border-slate-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-amber-500"
                    aria-label="قائمة المطاعم"
                ></textarea>
                <button
                    id="add-bulk-restaurants-btn"
                    class="action-btn w-full px-6 py-3 bg-amber-500 text-white font-bold rounded-lg hover:bg-amber-600 focus:outline-none focus:ring-2 focus:ring-amber-500 focus:ring-offset-2"
                >
                    أضف القائمة
                </button>
            </div>
            <button
                id="start-voting-btn"
                class="action-btn w-full px-6 py-3 bg-cyan-500 text-white font-bold rounded-lg hover:bg-cyan-600 focus:outline-none focus:ring-2 focus:ring-cyan-400 focus:ring-offset-2 mt-4 hidden"
            >
                ابدأ التصويت
            </button>
        </div>

        <div id="voting-section" class="space-y-4 mt-6 hidden">
            <h2 class="text-xl font-bold text-slate-700 border-b-2 border-slate-200 pb-2">3. صوّت!</h2>
            <div id="restaurants-list" class="space-y-4 max-h-80 overflow-y-auto scrollable-list pr-2">
                <p id="no-restaurants-message" class="text-center text-slate-500 italic">لم تتم إضافة أي مطاعم بعد. أضف بعضها أعلاه!</p>
            </div>
        </div>

        <div id="result-section" class="space-y-4 mt-6 p-4 bg-indigo-50 border border-indigo-200 rounded-xl shadow-inner hidden">
            <h2 id="result-title" class="text-xl font-bold text-slate-800 text-center"></h2>
            <p id="selected-restaurant" class="text-center text-3xl font-black text-indigo-600 py-4">
                </p>
            <div id="restart-options-div" class="flex flex-col gap-3 mt-4 hidden">
                <button
                    id="restart-with-same-restaurants-btn"
                    class="action-btn w-full px-6 py-3 bg-amber-500 text-white font-bold rounded-lg hover:bg-amber-600 focus:outline-none focus:ring-2 focus:ring-amber-500 focus:ring-offset-2"
                >
                    </button>
                <button
                    id="restart-all-btn"
                    class="action-btn w-full px-6 py-3 bg-rose-600 text-white font-bold rounded-lg hover:bg-rose-700 focus:outline-none focus:ring-2 focus:ring-rose-500 focus:ring-offset-2"
                >
                    بدء جلسة جديدة بالكامل
                </button>
            </div>
        </div>
    </div>

    <div id="custom-app-dialog" class="custom-dialog-backdrop hidden">
        <div class="custom-dialog-content">
            <p id="dialog-message" class="text-lg font-semibold text-slate-800"></p>
            <div class="flex justify-center gap-4 mt-4">
                <button id="dialog-ok-btn" class="px-5 py-2 bg-cyan-600 text-white rounded-lg hover:bg-cyan-700 transition">موافق</button>
                <button id="dialog-cancel-btn" class="px-5 py-2 bg-slate-300 text-slate-800 rounded-lg hover:bg-slate-400 transition hidden">إلغاء</button>
            </div>
        </div>
    </div>

    <script>
        // --- App States ---
        const APP_STATE = {
            JOINING_USERS: 'joining_users',
            ADDING_RESTAURANTS: 'adding_restaurants',
            VOTING: 'voting',
            RUNOFF_VOTING: 'runoff_voting',
            FINAL_RESULT: 'final_result'
        };

        // --- Global State Variables ---
        let currentAppState = APP_STATE.JOINING_USERS;
        let session = {
            users: [],
            restaurants: [],
            tiedRestaurantsForRunoff: [],
            selectedRestaurant: null
        };

        // --- DOM Elements ---
        const userNameInput = document.getElementById('user-name-input');
        const joinSessionBtn = document.getElementById('join-session-btn');
        const usersList = document.getElementById('users-list');
        const noUsersMessage = document.getElementById('no-users-message');

        const restaurantSection = document.getElementById('restaurant-section');
        const bulkRestaurantInput = document.getElementById('bulk-restaurant-input');
        const addBulkRestaurantsBtn = document.getElementById('add-bulk-restaurants-btn');
        const startVotingBtn = document.getElementById('start-voting-btn');

        const votingSection = document.getElementById('voting-section');
        const restaurantsListDiv = document.getElementById('restaurants-list');
        const noRestaurantsMessage = document.getElementById('no-restaurants-message');

        const resultSection = document.getElementById('result-section');
        const resultTitle = document.getElementById('result-title');
        const selectedRestaurantDisplay = document.getElementById('selected-restaurant');
        
        const restartOptionsDiv = document.getElementById('restart-options-div'); 
        const restartWithSameRestaurantsBtn = document.getElementById('restart-with-same-restaurants-btn'); 
        const restartAllBtn = document.getElementById('restart-all-btn'); 

        // --- Custom Dialog Function ---
        function showCustomDialog(message, type = 'info', onConfirm = null) {
            const dialog = document.getElementById('custom-app-dialog');
            const dialogMessage = document.getElementById('dialog-message');
            const dialogOkBtn = document.getElementById('dialog-ok-btn');
            const dialogCancelBtn = document.getElementById('dialog-cancel-btn');

            dialogMessage.textContent = message;

            dialogOkBtn.className = `px-5 py-2 rounded-lg transition text-white ${
                type === 'error' ? 'bg-rose-600 hover:bg-rose-700' :
                'bg-cyan-600 hover:bg-cyan-700'
            }`;

            dialogOkBtn.onclick = () => {
                dialog.classList.add('hidden');
                if (onConfirm) onConfirm(true);
            };

            if (onConfirm) {
                dialogCancelBtn.classList.remove('hidden');
                dialogCancelBtn.onclick = () => {
                    dialog.classList.add('hidden');
                    onConfirm(false);
                };
            } else {
                dialogCancelBtn.classList.add('hidden');
            }

            dialog.classList.remove('hidden');
        }
        
        // --- Auto-hiding Message Function ---
        function showAutoHidingMessage(message, type) {
            const existingMessage = document.getElementById('app-message');
            if (existingMessage) existingMessage.remove();

            const msgDiv = document.createElement('div');
            msgDiv.id = 'app-message';
            msgDiv.className = `p-3 rounded-lg text-sm text-center mb-4 ${
                type === 'success' ? 'bg-green-100 text-green-800' :
                type === 'error' ? 'bg-red-100 text-red-800' :
                'bg-blue-100 text-blue-800'
            }`;
            msgDiv.textContent = message;
            document.querySelector('.max-w-md').insertBefore(msgDiv, document.querySelector('.max-w-md').firstChild.nextSibling);

            setTimeout(() => {
                msgDiv.remove();
            }, 3000);
        }

        // --- UI State Management ---
        function updateUIBasedOnAppState() {
            document.querySelectorAll('#session-setup, #restaurant-section, #voting-section, #result-section').forEach(el => el.classList.add('hidden'));
            startVotingBtn.classList.add('hidden');
            restartOptionsDiv.classList.add('hidden');

            bulkRestaurantInput.disabled = false;
            addBulkRestaurantsBtn.disabled = false;
            userNameInput.disabled = false;
            joinSessionBtn.disabled = false;

            switch (currentAppState) {
                case APP_STATE.JOINING_USERS:
                    document.getElementById('session-setup').classList.remove('hidden');
                    break;

                case APP_STATE.ADDING_RESTAURANTS:
                    document.getElementById('session-setup').classList.remove('hidden');
                    restaurantSection.classList.remove('hidden');
                    if (session.users.length >= 2 && session.restaurants.length > 0) {
                        startVotingBtn.classList.remove('hidden');
                    }
                    break;

                case APP_STATE.VOTING:
                case APP_STATE.RUNOFF_VOTING:
                    document.getElementById('session-setup').classList.remove('hidden');
                    votingSection.classList.remove('hidden');
                    bulkRestaurantInput.disabled = true;
                    addBulkRestaurantsBtn.disabled = true;
                    break;

                case APP_STATE.FINAL_RESULT:
                    document.getElementById('session-setup').classList.remove('hidden');
                    resultSection.classList.remove('hidden');
                    bulkRestaurantInput.disabled = true;
                    addBulkRestaurantsBtn.disabled = true;
                    
                    restartOptionsDiv.classList.remove('hidden');
                    
                    if (session.selectedRestaurant) {
                        resultTitle.textContent = 'تم اتخاذ القرار! 🎉';
                        selectedRestaurantDisplay.textContent = session.selectedRestaurant;
                        restartWithSameRestaurantsBtn.textContent = 'إعادة التصويت';
                    } else {
                        resultTitle.textContent = 'لم يتم التوصل لاتفاق 😕';
                        selectedRestaurantDisplay.textContent = 'لا يوجد مطعم فائز بالإجماع.';
                        restartWithSameRestaurantsBtn.textContent = 'إعادة التصويت بنفس المطاعم';
                    }
                    restartAllBtn.textContent = 'بدء جلسة جديدة بالكامل';
                    break;
            }
            renderUsers();
            renderRestaurants();
        }

        // --- Helper Functions ---
        function generateUniqueId() { return crypto.randomUUID(); }

        function renderUsers() {
            usersList.innerHTML = '';
            if (session.users.length === 0) {
                noUsersMessage.style.display = 'block';
            } else {
                noUsersMessage.style.display = 'none';
                session.users.forEach(user => {
                    const li = document.createElement('li');
                    li.textContent = user.name;
                    usersList.appendChild(li);
                });
            }
        }

        function renderRestaurants() {
            restaurantsListDiv.innerHTML = '';
            let restaurantsToDisplay = (currentAppState === APP_STATE.RUNOFF_VOTING) ? session.tiedRestaurantsForRunoff : session.restaurants;

            if (currentAppState === APP_STATE.RUNOFF_VOTING) {
                restaurantsListDiv.innerHTML = '<p class="text-center text-slate-700 italic mb-4">جولة تصفية! صوّتوا بين الخيارات المتساوية:</p>';
            }

            if (restaurantsToDisplay.length === 0 && (currentAppState === APP_STATE.VOTING || currentAppState === APP_STATE.RUNOFF_VOTING)) {
                noRestaurantsMessage.style.display = 'block';
            } else {
                noRestaurantsMessage.style.display = 'none';
                restaurantsToDisplay.forEach(restaurant => {
                    const restaurantCard = document.createElement('div');
                    restaurantCard.id = `restaurant-${restaurant.id}`;
                    restaurantCard.className = 'bg-white p-4 rounded-lg shadow-md border border-slate-200';

                    let yesVotes = 0, noVotes = 0;
                    session.users.forEach(user => {
                        if (restaurant.votes[user.id] === true) yesVotes++;
                        else if (restaurant.votes[user.id] === false) noVotes++;
                    });

                    let progressMessage = '';
                    const totalUsers = session.users.length;
                    if (yesVotes === totalUsers && totalUsers > 0) {
                        progressMessage = '<span class="font-bold text-green-600">تم الاتفاق! ✅</span>';
                    } else if (noVotes > 0) {
                        progressMessage = `<span class="font-bold text-red-600">${noVotes} ${noVotes > 1 ? 'رفضوا' : 'رفض'} ❌</span>`;
                    } else if (yesVotes > 0) {
                        progressMessage = `<span class="text-amber-600">بانتظار ${totalUsers - yesVotes} أصوات...</span>`;
                    } else {
                        progressMessage = `لم يصوّت أحد بعد`;
                    }

                    restaurantCard.innerHTML = `
                        <div class="flex justify-between items-center mb-3">
                            <h3 class="text-lg font-bold text-slate-800">${restaurant.name}</h3>
                            <div class="text-sm text-slate-600 text-right">${progressMessage}</div>
                        </div>
                        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-${session.users.length > 2 ? '3' : '2'} gap-2">
                            ${session.users.map(user => `
                                <div class="flex flex-col items-center p-2 border rounded-lg ${restaurant.votes[user.id] === true ? 'bg-green-100 border-green-300' : restaurant.votes[user.id] === false ? 'bg-red-100 border-red-300' : 'bg-slate-50 border-slate-200'}">
                                    <span class="text-xs font-semibold text-slate-700 mb-1">${user.name}</span>
                                    <div class="flex gap-2">
                                        <button data-restaurant-id="${restaurant.id}" data-user-id="${user.id}" data-vote="yes" class="vote-btn p-2 rounded-full ${restaurant.votes[user.id] === true ? 'bg-green-500 hover:bg-green-600 scale-110' : 'bg-slate-300 hover:bg-slate-400'} text-white" ${(currentAppState !== APP_STATE.VOTING && currentAppState !== APP_STATE.RUNOFF_VOTING) ? 'disabled' : ''}>
                                            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd" /></svg>
                                        </button>
                                        <button data-restaurant-id="${restaurant.id}" data-user-id="${user.id}" data-vote="no" class="vote-btn p-2 rounded-full ${restaurant.votes[user.id] === false ? 'bg-red-500 hover:bg-red-600 scale-110' : 'bg-slate-300 hover:bg-slate-400'} text-white" ${(currentAppState !== APP_STATE.VOTING && currentAppState !== APP_STATE.RUNOFF_VOTING) ? 'disabled' : ''}>
                                            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd" /></svg>
                                        </button>
                                    </div>
                                </div>
                            `).join('')}
                        </div>`;
                    restaurantsListDiv.appendChild(restaurantCard);
                });
            }
            document.querySelectorAll('.vote-btn').forEach(button => button.onclick = handleVote);
        }

        function areAllVotesCast(restaurantsToConsider) {
            if (session.users.length === 0 || restaurantsToConsider.length === 0) return false;
            return restaurantsToConsider.every(r => session.users.every(u => r.votes[u.id] !== null));
        }

        function determineWinner(restaurantsToAnalyze) {
            const winners = restaurantsToAnalyze.filter(r => session.users.every(u => r.votes[u.id] === true));

            if (winners.length === 1) {
                session.selectedRestaurant = winners[0].name;
                currentAppState = APP_STATE.FINAL_RESULT;
                showAutoHidingMessage(`تم الاختيار: ${session.selectedRestaurant}!`, 'success');
            } else if (winners.length > 1) {
                session.tiedRestaurantsForRunoff = winners.map(r => ({ id: r.id, name: r.name, votes: {} }));
                session.tiedRestaurantsForRunoff.forEach(r => session.users.forEach(u => r.votes[u.id] = null));
                currentAppState = APP_STATE.RUNOFF_VOTING;
                showCustomDialog(`تعادل! جولة تصفية بين: ${winners.map(r => r.name).join(' و ')}`, 'info');
            } else {
                session.selectedRestaurant = null;
                currentAppState = APP_STATE.FINAL_RESULT;
                showCustomDialog('لم يتم الاتفاق على أي مطعم. يمكنكم إعادة المحاولة.', 'info');
            }
            updateUIBasedOnAppState();
        }

        // --- Event Handlers ---
        function handleJoinSession() {
            const userName = userNameInput.value.trim();
            if (!userName) {
                showAutoHidingMessage('الرجاء إدخال اسم للانضمام.', 'error');
                return;
            }
            if (session.users.find(u => u.name.toLowerCase() === userName.toLowerCase())) {
                showAutoHidingMessage('هذا الاسم مستخدم بالفعل.', 'info');
                return;
            }
            const newUser = { id: generateUniqueId(), name: userName };
            session.users.push(newUser);
            session.restaurants.forEach(r => r.votes[newUser.id] = null);
            showAutoHidingMessage(`أهلاً بك، ${userName}!`, 'success');
            userNameInput.value = '';
            if (session.users.length >= 2 && currentAppState === APP_STATE.JOINING_USERS) {
                currentAppState = APP_STATE.ADDING_RESTAURANTS;
            }
            updateUIBasedOnAppState();
        }

        function addRestaurantToSession(restaurantName) {
            if (session.restaurants.some(r => r.name.toLowerCase() === restaurantName.toLowerCase())) return false;
            const newRestaurant = { id: generateUniqueId(), name: restaurantName, votes: {} };
            session.users.forEach(user => newRestaurant.votes[user.id] = null);
            session.restaurants.push(newRestaurant);
            return true;
        }

        function handleAddBulkRestaurants() {
            const bulkText = bulkRestaurantInput.value.trim();
            if (!bulkText) {
                showAutoHidingMessage('الرجاء إدخال قائمة المطاعم.', 'error');
                return;
            }
            const names = bulkText.split(/[\n,]+/).map(name => name.trim()).filter(Boolean);
            let addedCount = 0;
            names.forEach(name => {
                if (addRestaurantToSession(name)) addedCount++;
            });
            if (addedCount > 0) {
                showAutoHidingMessage(`تمت إضافة ${addedCount} مطاعم!`, 'success');
                bulkRestaurantInput.value = '';
            } else {
                showAutoHidingMessage('لم يتم إضافة مطاعم جديدة (قد تكون مكررة).', 'info');
            }
            updateUIBasedOnAppState();
        }

        function handleStartVoting() {
            if (session.users.length < 2) {
                showCustomDialog('يلزم وجود شخصين على الأقل لبدء التصويت.', 'error');
                return;
            }
            if (session.restaurants.length === 0) {
                showCustomDialog('الرجاء إضافة مطاعم أولاً.', 'error');
                return;
            }
            currentAppState = APP_STATE.VOTING;
            showAutoHidingMessage('بدأ التصويت!', 'info');
            updateUIBasedOnAppState();
        }

        function handleVote(event) {
            const button = event.target.closest('button');
            if (!button) return;
            const { restaurantId, userId, vote } = button.dataset;
            let list = (currentAppState === APP_STATE.RUNOFF_VOTING) ? session.tiedRestaurantsForRunoff : session.restaurants;
            const restaurant = list.find(r => r.id === restaurantId);
            if (restaurant) {
                restaurant.votes[userId] = (vote === 'yes');
                renderRestaurants();
                if (areAllVotesCast(list)) {
                    determineWinner(list);
                }
            }
        }

        function resetRestaurantVotes(restaurantsToReset) {
            restaurantsToReset.forEach(r => session.users.forEach(u => r.votes[u.id] = null));
        }

        function handleRestartWithSameRestaurants() {
            showCustomDialog('هل تريد إعادة التصويت؟ سيتم مسح الأصوات الحالية.', 'info', confirmed => {
                if (confirmed) {
                    resetRestaurantVotes(session.restaurants);
                    session.tiedRestaurantsForRunoff = [];
                    session.selectedRestaurant = null;
                    currentAppState = APP_STATE.ADDING_RESTAURANTS;
                    showAutoHidingMessage('تمت إعادة التصويت. يمكنكم إضافة المزيد أو البدء من جديد.', 'info');
                    updateUIBasedOnAppState();
                }
            });
        }

        function handleRestartAll() {
            showCustomDialog('هل تريد بدء جلسة جديدة بالكامل؟ سيتم مسح كل البيانات.', 'error', confirmed => {
                if (confirmed) {
                    session = { users: [], restaurants: [], tiedRestaurantsForRunoff: [], selectedRestaurant: null };
                    userNameInput.value = '';
                    bulkRestaurantInput.value = '';
                    currentAppState = APP_STATE.JOINING_USERS;
                    showAutoHidingMessage('تم بدء جلسة جديدة!', 'info');
                    updateUIBasedOnAppState();
                }
            });
        }

        // --- Initial Setup & Event Listeners ---
        document.addEventListener('DOMContentLoaded', () => {
            updateUIBasedOnAppState();
            joinSessionBtn.addEventListener('click', handleJoinSession);
            addBulkRestaurantsBtn.addEventListener('click', handleAddBulkRestaurants);
            startVotingBtn.addEventListener('click', handleStartVoting);
            restartWithSameRestaurantsBtn.addEventListener('click', handleRestartWithSameRestaurants);
            restartAllBtn.addEventListener('click', handleRestartAll);
            userNameInput.addEventListener('keypress', e => { if (e.key === 'Enter') handleJoinSession(); });
            bulkRestaurantInput.addEventListener('keypress', e => { if (e.key === 'Enter' && (e.ctrlKey || e.metaKey)) { handleAddBulkRestaurants(); e.preventDefault(); } });
        });
    </script>
    <footer class="mt-8 text-xs text-center text-slate-400">
  صنع بواسطة عمك يزيد
</footer>
</body>
</html>
