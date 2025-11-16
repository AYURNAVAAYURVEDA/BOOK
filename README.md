<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Medical Center Booking System</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Use Inter font family -->
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f7f9fb; }
        .tab-button.active { background-color: #10b981; color: white; }
    </style>
</head>
<body class="p-4 md:p-8 min-h-screen">

    <!-- Header and View Toggles -->
    <header class="mb-8 flex flex-col sm:flex-row justify-between items-center bg-white p-4 rounded-xl shadow-lg">
        <h1 class="text-3xl font-bold text-gray-800 mb-4 sm:mb-0">Medical Booking Dashboard</h1>
        <div class="flex space-x-2 p-1 bg-gray-100 rounded-lg">
            <button id="patient-tab" class="tab-button active px-4 py-2 text-sm font-medium rounded-lg transition" onclick="switchView('patient')">
                Patient Booking
            </button>
            <button id="monitor-tab" class="tab-button px-4 py-2 text-sm font-medium rounded-lg transition" onclick="switchView('monitor')">
                Doctor Monitor
            </button>
        </div>
    </header>

    <!-- Authentication Status and User ID -->
    <div id="auth-status" class="mb-6 p-3 bg-indigo-100 text-indigo-800 rounded-lg shadow-md text-sm">
        Authenticating...
    </div>

    <!-- Main Content Area -->
    <main class="max-w-6xl mx-auto">
        <!-- Patient Booking View (Form) -->
        <section id="patient-view" class="bg-white p-6 md:p-10 rounded-xl shadow-2xl">
            <h2 class="text-2xl font-semibold mb-6 text-emerald-600 border-b pb-2">Book Your Appointment</h2>

            <form id="booking-form" class="grid grid-cols-1 md:grid-cols-2 gap-6">

                <!-- Date Selection -->
                <div class="md:col-span-2">
                    <label class="block text-gray-700 font-medium mb-2">Select Date (Next 7 Days)</label>
                    <div id="date-selection" class="flex flex-wrap gap-2">
                        <!-- Dates will be populated by JavaScript -->
                    </div>
                    <input type="hidden" id="selected-date" required>
                </div>

                <!-- Time Selection -->
                <div class="md:col-span-2">
                    <label class="block text-gray-700 font-medium mb-2">Select Time Slot</label>
                    <div id="time-selection" class="flex flex-wrap gap-2">
                        <!-- Time slots will be populated by JavaScript -->
                    </div>
                    <input type="hidden" id="selected-time" required>
                </div>

                <!-- Patient Name -->
                <div>
                    <label for="patient-name" class="block text-gray-700 font-medium mb-2">Patient Name</label>
                    <input type="text" id="patient-name" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-emerald-500 focus:border-emerald-500" placeholder="e.g., Jane Doe">
                </div>

                <!-- ID Number -->
                <div>
                    <label for="id-number" class="block text-gray-700 font-medium mb-2">ID Number</label>
                    <input type="text" id="id-number" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-emerald-500 focus:border-emerald-500" placeholder="e.g., 9876543210">
                </div>

                <!-- Mobile Number -->
                <div class="md:col-span-2">
                    <label for="mobile-number" class="block text-gray-700 font-medium mb-2">Mobile Number</label>
                    <input type="tel" id="mobile-number" required class="w-full p-3 border border-gray-300 rounded-lg focus:ring-emerald-500 focus:border-emerald-500" placeholder="e.g., +1234567890">
                </div>

                <!-- Submit and Status -->
                <div class="md:col-span-2">
                    <button type="submit" id="book-button" class="w-full bg-emerald-500 hover:bg-emerald-600 text-white font-bold py-3 px-4 rounded-lg transition duration-200 shadow-md">
                        Book Now
                    </button>
                </div>
            </form>

            <!-- Confirmation Message Modal -->
            <div id="confirmation-modal" class="fixed inset-0 bg-gray-900 bg-opacity-50 hidden items-center justify-center p-4">
                <div class="bg-white p-8 rounded-xl shadow-2xl max-w-sm w-full text-center transform scale-95 transition-transform duration-300 ease-out">
                    <svg class="w-16 h-16 text-emerald-500 mx-auto mb-4" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
                    <h3 class="text-xl font-semibold text-gray-800 mb-2">Booking Confirmed!</h3>
                    <p class="text-gray-600 mb-6">Your appointment has been successfully scheduled. We look forward to seeing you!</p>
                    <button id="close-modal" class="bg-emerald-500 hover:bg-emerald-600 text-white font-bold py-2 px-4 rounded-lg transition">
                        Close
                    </button>
                </div>
            </div>

        </section>

        <!-- Doctor Monitor View (List) -->
        <section id="monitor-view" class="bg-white p-6 md:p-10 rounded-xl shadow-2xl hidden">
            <h2 class="text-2xl font-semibold mb-6 text-indigo-600 border-b pb-2">Doctor Appointment Monitor</h2>

            <div id="booking-list" class="space-y-4">
                <div class="text-gray-500 text-center py-8" id="loading-indicator">Loading appointments...</div>
                <!-- Bookings will be dynamically inserted here -->
            </div>
            <div id="no-bookings" class="hidden text-center text-gray-500 py-12 border border-dashed rounded-lg mt-4">
                <p class="text-lg font-medium">No appointments scheduled yet.</p>
                <p class="text-sm">Check the Patient Booking tab to add a new appointment.</p>
            </div>
        </section>
    </main>

    <!-- Firebase Imports and Script -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged, setPersistence, browserLocalPersistence } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, addDoc, onSnapshot, collection, query, orderBy, setDoc, deleteDoc, serverTimestamp, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global Firebase Variables (Provided by Canvas Environment)
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        let db, auth, userId = null;
        let isAuthReady = false;

        // UI elements
        const patientView = document.getElementById('patient-view');
        const monitorView = document.getElementById('monitor-view');
        const patientTab = document.getElementById('patient-tab');
        const monitorTab = document.getElementById('monitor-tab');
        const authStatus = document.getElementById('auth-status');
        const dateSelectionDiv = document.getElementById('date-selection');
        const timeSelectionDiv = document.getElementById('time-selection');
        const selectedDateInput = document.getElementById('selected-date');
        const selectedTimeInput = document.getElementById('selected-time');
        const bookingListDiv = document.getElementById('booking-list');
        const confirmationModal = document.getElementById('confirmation-modal');
        const closeModalBtn = document.getElementById('close-modal');
        const noBookingsMessage = document.getElementById('no-bookings');
        const loadingIndicator = document.getElementById('loading-indicator');

        // Available Time Slots
        const timeSlots = [
            "09:00 AM", "10:00 AM", "11:00 AM", "02:00 PM", "03:00 PM", "04:00 PM", "05:00 PM"
        ];

        // --- Core Functions ---

        // Helper function to format date objects
        const formatDate = (date) => {
            return date.toLocaleDateString('en-US', { weekday: 'short', month: 'short', day: 'numeric' });
        };

        // Populate Date and Time selectors
        function populateSelectors() {
            // 1. Dates (Today + 6 days = 7 days total)
            const dates = [];
            const today = new Date();
            for (let i = 0; i < 7; i++) {
                const date = new Date(today);
                date.setDate(today.getDate() + i);
                dates.push({ label: formatDate(date), value: date.toISOString().split('T')[0] });
            }

            dateSelectionDiv.innerHTML = dates.map(date => `
                <button type="button" data-value="${date.value}" class="date-button px-4 py-2 bg-gray-200 text-gray-700 text-sm font-medium rounded-lg hover:bg-emerald-100 transition">
                    ${date.label}
                </button>
            `).join('');

            // 2. Times
            timeSelectionDiv.innerHTML = timeSlots.map(time => `
                <button type="button" data-value="${time}" class="time-button px-4 py-2 bg-gray-200 text-gray-700 text-sm font-medium rounded-lg hover:bg-emerald-100 transition">
                    ${time}
                </button>
            `).join('');

            // Add event listeners for dynamic selection
            addSelectionListeners('date');
            addSelectionListeners('time');

            // Select the first date and time by default
            if (dates.length > 0) {
                document.querySelector('.date-button').click();
            }
            if (timeSlots.length > 0) {
                document.querySelector('.time-button').click();
            }
        }

        function addSelectionListeners(type) {
            const container = type === 'date' ? dateSelectionDiv : timeSelectionDiv;
            const input = type === 'date' ? selectedDateInput : selectedTimeInput;
            const buttons = container.querySelectorAll(`.${type}-button`);

            buttons.forEach(button => {
                button.addEventListener('click', () => {
                    // Deselect all others
                    buttons.forEach(b => b.classList.remove('bg-emerald-500', 'text-white'));
                    // Select current
                    button.classList.add('bg-emerald-500', 'text-white');
                    // Update hidden input field
                    input.value = button.dataset.value;
                });
            });
        }


        function switchView(view) {
            if (view === 'patient') {
                patientView.classList.remove('hidden');
                monitorView.classList.add('hidden');
                patientTab.classList.add('active');
                monitorTab.classList.remove('active');
            } else {
                patientView.classList.add('hidden');
                monitorView.classList.remove('hidden');
                patientTab.classList.remove('active');
                monitorTab.classList.add('active');
                // Ensure monitor view data is listening for updates
                if (isAuthReady) {
                    subscribeToBookings();
                }
            }
        }

        // --- Firebase Initialization and Auth ---

        async function initializeFirebase() {
            try {
                if (!firebaseConfig) {
                     authStatus.textContent = 'ERROR: Firebase config is missing.';
                     return;
                }
                setLogLevel('debug');
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                // Try to set persistence (optional, but good practice)
                await setPersistence(auth, browserLocalPersistence).catch(error => {
                    console.error("Persistence setup failed, proceeding anyway:", error);
                });

                // Sign in with custom token or anonymously
                if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken);
                } else {
                    await signInAnonymously(auth);
                }

                // Wait for auth state to be ready
                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        userId = user.uid;
                        isAuthReady = true;
                        authStatus.classList.remove('bg-indigo-100');
                        authStatus.classList.add('bg-green-100');
                        authStatus.textContent = `Authenticated. User ID: ${userId}`;

                        // Start real-time listening if doctor view is active
                        if (!monitorView.classList.contains('hidden')) {
                             subscribeToBookings();
                        }
                    } else {
                        isAuthReady = true; // Still ready, but anonymous
                        userId = crypto.randomUUID();
                        authStatus.classList.add('bg-yellow-100');
                        authStatus.textContent = `Signed in Anonymously. Data will be saved under temporary ID: ${userId}`;
                    }
                });

            } catch (error) {
                console.error("Firebase Initialization Error:", error);
                authStatus.textContent = `ERROR: Firebase setup failed. ${error.message}`;
            }
        }


        // --- Firestore Operations ---

        // Public collection path for shared bookings data
        function getBookingsCollectionRef() {
            if (!db) throw new Error("Firestore not initialized.");
            // Public path: /artifacts/{appId}/public/data/bookings
            return collection(db, 'artifacts', appId, 'public', 'data', 'bookings');
        }

        async function handleBookingSubmit(event) {
            event.preventDefault();
            if (!isAuthReady || !db) {
                alert('System is still initializing. Please wait a moment.');
                return;
            }

            const button = document.getElementById('book-button');
            button.disabled = true;
            button.textContent = 'Booking...';

            try {
                const bookingData = {
                    date: selectedDateInput.value,
                    time: selectedTimeInput.value,
                    patientName: document.getElementById('patient-name').value,
                    idNumber: document.getElementById('id-number').value,
                    mobileNumber: document.getElementById('mobile-number').value,
                    bookedAt: serverTimestamp(),
                    userId: userId, // Record which user made the booking
                    status: 'PENDING'
                };

                // Add document to the public collection
                await addDoc(getBookingsCollectionRef(), bookingData);

                // Show confirmation message
                confirmationModal.classList.remove('hidden');
                confirmationModal.classList.add('flex');

                // Reset form fields
                document.getElementById('booking-form').reset();
                // Re-select default date/time visuals
                populateSelectors();

            } catch (error) {
                console.error("Error adding document: ", error);
                // Simple error display using the modal area
                alert('Booking failed. Please check the console for details.');
            } finally {
                button.disabled = false;
                button.textContent = 'Book Now';
            }
        }

        let unsubscribeBookings = null;

        function subscribeToBookings() {
            if (!isAuthReady || !db) return;

            // Stop previous listener if it exists
            if (unsubscribeBookings) {
                unsubscribeBookings();
            }

            // Create a query for the bookings collection, ordered by date and time
            const bookingsQuery = query(getBookingsCollectionRef(), orderBy('bookedAt', 'desc'));

            // Start real-time listener
            unsubscribeBookings = onSnapshot(bookingsQuery, (snapshot) => {
                const bookings = [];
                snapshot.forEach(doc => {
                    const data = doc.data();
                    bookings.push({ id: doc.id, ...data });
                });
                renderBookings(bookings);
            }, (error) => {
                console.error("Error listening to bookings: ", error);
                bookingListDiv.innerHTML = `<p class="text-red-500 text-center">Error loading appointments: ${error.message}</p>`;
            });
        }

        // --- Rendering Logic for Monitor View ---

        function renderBookings(bookings) {
            loadingIndicator.classList.add('hidden');
            bookingListDiv.innerHTML = ''; // Clear existing list

            if (bookings.length === 0) {
                noBookingsMessage.classList.remove('hidden');
                return;
            }
            noBookingsMessage.classList.add('hidden');

            bookings.forEach(booking => {
                const bookedDate = new Date(booking.bookedAt?.toDate());
                const formattedDate = booking.bookedAt ? bookedDate.toLocaleString() : 'N/A';
                
                const card = document.createElement('div');
                card.className = 'bg-gray-50 p-4 border border-gray-200 rounded-xl shadow-md transition hover:shadow-lg';
                card.innerHTML = `
                    <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center border-b pb-2 mb-2">
                        <h3 class="text-lg font-bold text-indigo-700">${booking.patientName}</h3>
                        <span class="px-3 py-1 text-xs font-semibold rounded-full bg-blue-100 text-blue-800">${booking.status}</span>
                    </div>
                    <div class="grid grid-cols-2 gap-2 text-sm text-gray-600">
                        <p><span class="font-medium text-gray-800">Date:</span> ${booking.date}</p>
                        <p><span class="font-medium text-gray-800">Time:</span> ${booking.time}</p>
                        <p><span class="font-medium text-gray-800">ID No:</span> ${booking.idNumber}</p>
                        <p><span class="font-medium text-gray-800">Mobile:</span> ${booking.mobileNumber}</p>
                        <p class="col-span-2 text-xs text-gray-400">Booked At: ${formattedDate}</p>
                    </div>
                `;
                bookingListDiv.appendChild(card);
            });
        }

        // --- Event Listeners and Initial Setup ---

        // Initialize App on load
        window.onload = () => {
            if (firebaseConfig) {
                initializeFirebase();
            } else {
                authStatus.textContent = 'WARNING: Firebase config is missing. Data will not be saved.';
                isAuthReady = true; // Allow testing form even without persistence
            }
            populateSelectors();
        };

        // Attach form submit handler
        document.getElementById('booking-form').addEventListener('submit', handleBookingSubmit);

        // Close modal handler
        closeModalBtn.addEventListener('click', () => {
            confirmationModal.classList.add('hidden');
            confirmationModal.classList.remove('flex');
        });

        // Expose function globally for the onclick attribute
        window.switchView = switchView;
    </script>

</body>
</html>
