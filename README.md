<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <meta name="theme-color" content="#0a0e27">
    <meta name="description" content="Task & Earn Pro - Complete tasks, earn money, withdraw to bKash, Nagad, Rocket.">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>Task & Earn Pro</title>

    <!-- PWA Manifest (dynamically generated) -->
    <link rel="manifest" href="data:application/manifest+json,PLACEHOLDER">

    <!-- Firebase SDK (v10 Modular via CDN) -->
    <script type="module">
        import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.13.1/firebase-app.js';
        import { getAuth, onAuthStateChanged, signInWithEmailAndPassword, createUserWithEmailAndPassword, signInWithPopup, GoogleAuthProvider, sendPasswordResetEmail, signOut, updateProfile, sendEmailVerification } from 'https://www.gstatic.com/firebasejs/10.13.1/firebase-auth.js';
        import { getDatabase, ref, set, get, push, update, remove, onValue, query, orderByChild, equalTo, serverTimestamp, increment, runTransaction } from 'https://www.gstatic.com/firebasejs/10.13.1/firebase-database.js';

        // ========================
        // 1. FIREBASE CONFIGURATION
        // ========================
        const firebaseConfig = {
            apiKey: "AIzaSyCbpH21fmOx2svCaTeiahzkeAJMyYdXfSA",
            authDomain: "sweet-26d35.firebaseapp.com",
            databaseURL: "https://sweet-26d35-default-rtdb.firebaseio.com",
            projectId: "sweet-26d35",
            storageBucket: "sweet-26d35.firebasestorage.app",
            messagingSenderId: "67161052210",
            appId: "1:67161052210:web:e2d9d8727c1aeb0e84d82b",
            measurementId: "G-X054X2XSS8"
        };

        // Initialize Firebase
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getDatabase(app);
        const googleProvider = new GoogleAuthProvider();

        // Make available globally for inline handlers
        window.auth = auth;
        window.db = db;
        window.firebaseFns = {
            ref, set, get, push, update, remove, onValue, query, orderByChild, equalTo,
            serverTimestamp, increment, runTransaction,
            onAuthStateChanged, signInWithEmailAndPassword, createUserWithEmailAndPassword,
            signInWithPopup, sendPasswordResetEmail, signOut, updateProfile, sendEmailVerification,
            GoogleAuthProvider
        };

        // ========================
        // 2. ADMIN CREDENTIALS
        // ========================
        const ADMIN_EMAIL = "niloychowdhury724@gmail.com";
        const ADMIN_PASSWORD = "Sahalam09";
        const ADMIN_UID = "5zOw2EjcofO9FZM2euqNvY2jOq52"; // Set after first admin login

        // ========================
        // 3. APP STATE
        // ========================
        const AppState = {
            user: null,
            userData: null,
            isAdmin: false,
            currentPage: 'home',
            previousPage: 'home',
            withdrawals: [],
            tasks: [],
            notifications: [],
            settings: {
                minWithdraw: 500,
                coinRate: 1, // 10000 coin = 1 BDT
                referralBonus: 50,
                dailyBonusBase: 10,
                spinRewardMin: 5,
                spinRewardMax: 50,
                appName: "Task & Earn Pro",
                supportEmail: "sahalamsikder14@gmail.com",
                bkashNumber: "01XXXXXXXXX",
                nagadNumber: "01XXXXXXXXX",
                rocketNumber: "01XXXXXXXXX"
            },
            ads: {
                banner: '',
                interstitial: '',
                rewarded: ''
            }
        };

        // Make functions available globally
        window.appState = AppState;
        window.showToast = showToast;
        window.showLoading = showLoading;
        window.hideLoading = hideLoading;
        window.navigateTo = navigateTo;
        window.handleLogin = handleLogin;
        window.handleRegister = handleRegister;
        window.handleGoogleLogin = handleGoogleLogin;
        window.handleForgotPassword = handleForgotPassword;
        window.handleLogout = handleLogout;
        window.claimDailyBonus = claimDailyBonus;
        window.claimReferralBonus = claimReferralBonus;
        window.spinWheel = spinWheel;
        window.scratchCard = scratchCard;
        window.completeTask = completeTask;
        window.submitWithdraw = submitWithdraw;
        window.copyReferralLink = copyReferralLink;
        window.shareReferralLink = shareReferralLink;
        window.adminLogin = adminLogin;
        window.adminLogout = adminLogout;
        window.adminApproveWithdraw = adminApproveWithdraw;
        window.adminRejectWithdraw = adminRejectWithdraw;
        window.adminBanUser = adminBanUser;
        window.adminAddCoins = adminAddCoins;
        window.adminRemoveCoins = adminRemoveCoins;
        window.adminCreateTask = adminCreateTask;
        window.adminDeleteTask = adminDeleteTask;
        window.adminSendNotification = adminSendNotification;
        window.adminUpdateSettings = adminUpdateSettings;
        window.toggleSidebar = toggleSidebar;
        window.closeModal = closeModal;
        window.openModal = openModal;
        window.watchAd = watchAd;

        // ========================
        // 4. UI UTILITIES
        // ========================
        function showToast(message, type = 'info') {
            const container = document.getElementById('toastContainer');
            const toast = document.createElement('div');
            toast.className = `toast toast-${type}`;
            toast.innerHTML = `
                <span class="toast-icon">${type === 'success' ? '✓' : type === 'error' ? '✕' : 'ℹ'}</span>
                <span class="toast-message">${message}</span>
            `;
            container.appendChild(toast);
            setTimeout(() => {
                toast.style.animation = 'slideOutRight 0.3s ease forwards';
                setTimeout(() => toast.remove(), 300);
            }, 3000);
        }

        function showLoading(text = 'Loading...') {
            const loader = document.getElementById('loadingOverlay');
            document.getElementById('loadingText').textContent = text;
            loader.classList.add('active');
        }

        function hideLoading() {
            document.getElementById('loadingOverlay').classList.remove('active');
        }

        function formatCurrency(amount) {
            return '৳' + parseFloat(amount || 0).toFixed(2);
        }

        function formatDate(timestamp) {
            if (!timestamp) return 'N/A';
            const date = new Date(timestamp);
            return date.toLocaleDateString('en-US', { year: 'numeric', month: 'short', day: 'numeric' });
        }

        function formatDateTime(timestamp) {
            if (!timestamp) return 'N/A';
            const date = new Date(timestamp);
            return date.toLocaleString('en-US', { month: 'short', day: 'numeric', hour: '2-digit', minute: '2-digit' });
        }

        function generateReferralCode() {
            return 'TE' + Math.random().toString(36).substring(2, 8).toUpperCase();
        }

        function validateEmail(email) {
            return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
        }

        function validatePhone(phone) {
            return /^01[3-9]\d{8}$/.test(phone);
        }

        function sanitizeInput(input) {
            const div = document.createElement('div');
            div.textContent = input;
            return div.innerHTML;
        }

        // ========================
        // 5. NAVIGATION
        // ========================
        function navigateTo(page) {
            AppState.previousPage = AppState.currentPage;
            AppState.currentPage = page;

            // Update bottom nav
            document.querySelectorAll('.nav-item').forEach(item => {
                item.classList.toggle('active', item.dataset.page === page);
            });

            // Update pages
            document.querySelectorAll('.page').forEach(p => {
                p.classList.toggle('active', p.id === `page-${page}`);
            });

            // Update header
            const titles = {
                home: 'Dashboard',
                earn: 'Earn Money',
                wallet: 'My Wallet',
                withdraw: 'Withdraw',
                referrals: 'Referrals',
                notifications: 'Notifications',
                profile: 'Profile',
                support: 'Support',
                privacy: 'Privacy Policy',
                terms: 'Terms & Conditions',
                admin: 'Admin Panel'
            };
            document.getElementById('headerTitle').textContent = titles[page] || 'Task & Earn Pro';

            // Scroll to top
            window.scrollTo({ top: 0, behavior: 'smooth' });

            // Load page-specific data
            if (page === 'home') loadHomeData();
            if (page === 'wallet') loadWalletData();
            if (page === 'withdraw') loadWithdrawPage();
            if (page === 'referrals') loadReferralData();
            if (page === 'notifications') loadNotifications();
            if (page === 'profile') loadProfileData();
            if (page === 'admin') loadAdminDashboard();
            if (page === 'earn') loadEarnTasks();
        }

        function toggleSidebar() {
            const sidebar = document.getElementById('sidebar');
            const overlay = document.getElementById('sidebarOverlay');
            sidebar.classList.toggle('open');
            overlay.classList.toggle('active');
        }

        function openModal(title, content, actions = '') {
            document.getElementById('modalTitle').textContent = title;
            document.getElementById('modalBody').innerHTML = content;
            document.getElementById('modalActions').innerHTML = actions;
            document.getElementById('modalOverlay').classList.add('active');
        }

        function closeModal() {
            document.getElementById('modalOverlay').classList.remove('active');
        }

        // ========================
        // 6. AUTHENTICATION
        // ========================
        async function handleLogin(email, password) {
            if (!validateEmail(email)) { showToast('Please enter a valid email', 'error'); return; }
            if (password.length < 6) { showToast('Password must be at least 6 characters', 'error'); return; }
            showLoading('Signing in...');
            try {
                await firebaseFns.signInWithEmailAndPassword(auth, email, password);
                showToast('Welcome back!', 'success');
            } catch (error) {
                showToast(error.message, 'error');
            } finally {
                hideLoading();
            }
        }

        async function handleRegister(name, email, password, referralCode) {
            if (!name || name.length < 2) { showToast('Please enter your full name', 'error'); return; }
            if (!validateEmail(email)) { showToast('Please enter a valid email', 'error'); return; }
            if (password.length < 6) { showToast('Password must be at least 6 characters', 'error'); return; }
            showLoading('Creating account...');
            try {
                const userCredential = await firebaseFns.createUserWithEmailAndPassword(auth, email, password);
                const user = userCredential.user;
                await firebaseFns.updateProfile(user, { displayName: name });

                // Check if referral code exists
                let referredBy = null;
                if (referralCode) {
                    const usersRef = firebaseFns.ref(db, 'users');
                    const snapshot = await firebaseFns.get(firebaseFns.query(usersRef, firebaseFns.orderByChild('referralCode'), firebaseFns.equalTo(referralCode.toUpperCase())));
                    if (snapshot.exists()) {
                        const users = snapshot.val();
                        referredBy = Object.keys(users)[0];
                    }
                }

                // Create user record in database
                const newReferralCode = generateReferralCode();
                await firebaseFns.set(firebaseFns.ref(db, `users/${user.uid}`), {
                    uid: user.uid,
                    name: name,
                    email: email,
                    photo: user.photoURL || '',
                    balance: 0,
                    withdrawableBalance: 0,
                    totalEarnings: 0,
                    referralEarnings: 0,
                    todayEarnings: 0,
                    referralCode: newReferralCode,
                    referredBy: referredBy || '',
                    referralCount: 0,
                    rank: 'Bronze',
                    joinDate: Date.now(),
                    lastLogin: Date.now(),
                    lastBonusClaim: 0,
                    lastSpinTime: 0,
                    lastScratchTime: 0,
                    lastCheckIn: 0,
                    checkInStreak: 0,
                    completedTasks: {},
                    banned: false,
                    isAdmin: false,
                    spinCount: 0,
                    scratchCount: 0,
                    withdrawalCount: 0,
                    referralBonusClaimed: false
                });

                // If referred, update referrer
                if (referredBy) {
                    const referrerRef = firebaseFns.ref(db, `users/${referredBy}`);
                    await firebaseFns.update(referrerRef, {
                        referralCount: firebaseFns.increment(1),
                        balance: firebaseFns.increment(AppState.settings.referralBonus),
                        referralEarnings: firebaseFns.increment(AppState.settings.referralBonus),
                        totalEarnings: firebaseFns.increment(AppState.settings.referralBonus)
                    });

                    // Add referral record
                    await firebaseFns.push(firebaseFns.ref(db, 'referrals'), {
                        referrerId: referredBy,
                        referredId: user.uid,
                        referredName: name,
                        bonus: AppState.settings.referralBonus,
                        date: Date.now()
                    });

                    // Add transaction for referrer
                    await firebaseFns.push(firebaseFns.ref(db, `transactions/${referredBy}`), {
                        type: 'referral_bonus',
                        amount: AppState.settings.referralBonus,
                        description: `Referral bonus from ${name}`,
                        date: Date.now(),
                        status: 'completed'
                    });
                }

                await firebaseFns.sendEmailVerification(user);
                showToast('Account created! Please verify your email.', 'success');
            } catch (error) {
                if (error.code === 'auth/email-already-in-use') {
                    showToast('This email is already registered', 'error');
                } else {
                    showToast(error.message, 'error');
                }
            } finally {
                hideLoading();
            }
        }

        async function handleGoogleLogin() {
            showLoading('Signing in with Google...');
            try {
                const result = await firebaseFns.signInWithPopup(auth, googleProvider);
                const user = result.user;

                // Check if user exists in database
                const userRef = firebaseFns.ref(db, `users/${user.uid}`);
                const snapshot = await firebaseFns.get(userRef);

                if (!snapshot.exists()) {
                    const newReferralCode = generateReferralCode();
                    await firebaseFns.set(userRef, {
                        uid: user.uid,
                        name: user.displayName || 'User',
                        email: user.email,
                        photo: user.photoURL || '',
                        balance: 0,
                        withdrawableBalance: 0,
                        totalEarnings: 0,
                        referralEarnings: 0,
                        todayEarnings: 0,
                        referralCode: newReferralCode,
                        referredBy: '',
                        referralCount: 0,
                        rank: 'Bronze',
                        joinDate: Date.now(),
                        lastLogin: Date.now(),
                        lastBonusClaim: 0,
                        lastSpinTime: 0,
                        lastScratchTime: 0,
                        lastCheckIn: 0,
                        checkInStreak: 0,
                        completedTasks: {},
                        banned: false,
                        isAdmin: false,
                        spinCount: 0,
                        scratchCount: 0,
                        withdrawalCount: 0,
                        referralBonusClaimed: false
                    });
                } else {
                    await firebaseFns.update(userRef, { lastLogin: Date.now() });
                }

                showToast('Signed in successfully!', 'success');
            } catch (error) {
                showToast(error.message, 'error');
            } finally {
                hideLoading();
            }
        }

        async function handleForgotPassword(email) {
            if (!validateEmail(email)) { showToast('Please enter a valid email', 'error'); return; }
            showLoading('Sending reset link...');
            try {
                await firebaseFns.sendPasswordResetEmail(auth, email);
                showToast('Password reset link sent to your email', 'success');
                closeModal();
            } catch (error) {
                showToast(error.message, 'error');
            } finally {
                hideLoading();
            }
        }

        async function handleLogout() {
            showLoading('Signing out...');
            try {
                await firebaseFns.signOut(auth);
                AppState.user = null;
                AppState.userData = null;
                AppState.isAdmin = false;
                showToast('Signed out', 'info');
            } catch (error) {
                showToast(error.message, 'error');
            } finally {
                hideLoading();
            }
        }

        // ========================
        // 7. AUTH STATE OBSERVER
        // ========================
        firebaseFns.onAuthStateChanged(auth, async (user) => {
            if (user) {
                AppState.user = user;

                // Check if admin
                if (user.email === ADMIN_EMAIL) {
                    AppState.isAdmin = true;
                }

                // Load user data
                const userRef = firebaseFns.ref(db, `users/${user.uid}`);
                firebaseFns.onValue(userRef, (snapshot) => {
                    if (snapshot.exists()) {
                        AppState.userData = snapshot.val();
                        if (AppState.userData.banned) {
                            showToast('Your account has been banned. Contact support.', 'error');
                            firebaseFns.signOut(auth);
                            return;
                        }
                        updateUI();
                    }
                });

                // Load settings
                const settingsRef = firebaseFns.ref(db, 'settings');
                firebaseFns.onValue(settingsRef, (snapshot) => {
                    if (snapshot.exists()) {
                        AppState.settings = { ...AppState.settings, ...snapshot.val() };
                    }
                });

                // Load ads
                const adsRef = firebaseFns.ref(db, 'ads');
                firebaseFns.onValue(adsRef, (snapshot) => {
                    if (snapshot.exists()) {
                        AppState.ads = { ...AppState.ads, ...snapshot.val() };
                        loadBannerAd();
                    }
                });

                showApp();
            } else {
                AppState.user = null;
                AppState.userData = null;
                AppState.isAdmin = false;
                showAuth();
            }
        });

        function showApp() {
            document.getElementById('authScreen').style.display = 'none';
            document.getElementById('appScreen').style.display = 'flex';
            updateUI();
            navigateTo('home');
        }

        function showAuth() {
            document.getElementById('authScreen').style.display = 'flex';
            document.getElementById('appScreen').style.display = 'none';
            renderAuthScreen();
        }

        // ========================
        // 8. UI UPDATES
        // ========================
        function updateUI() {
            if (!AppState.userData) return;
            const d = AppState.userData;

            // Header
            document.getElementById('headerBalance').textContent = formatCurrency(d.balance);
            document.getElementById('headerAvatar').src = d.photo || `https://ui-avatars.com/api/?name=${encodeURIComponent(d.name)}&background=3b82f6&color=fff&size=80`;

            // Home page
            document.getElementById('homeBalance').textContent = formatCurrency(d.balance);
            document.getElementById('homeTotalEarnings').textContent = formatCurrency(d.totalEarnings);
            document.getElementById('homeReferralEarnings').textContent = formatCurrency(d.referralEarnings);
            document.getElementById('homeTodayEarnings').textContent = formatCurrency(d.todayEarnings);
            document.getElementById('homeRank').textContent = d.rank || 'Bronze';
            document.getElementById('homeUserName').textContent = d.name;
            document.getElementById('homeUserEmail').textContent = d.email;
            document.getElementById('homeAvatar').src = d.photo || `https://ui-avatars.com/api/?name=${encodeURIComponent(d.name)}&background=3b82f6&color=fff&size=120`;

            // Referral
            document.getElementById('referralCode').textContent = d.referralCode;
            document.getElementById('referralCount').textContent = d.referralCount || 0;
            document.getElementById('referralEarningsDisplay').textContent = formatCurrency(d.referralEarnings);
            const refLink = `${window.location.origin}${window.location.pathname}?ref=${d.referralCode}`;
            document.getElementById('referralLink').value = refLink;

            // Profile
            document.getElementById('profileName').textContent = d.name;
            document.getElementById('profileEmail').textContent = d.email;
            document.getElementById('profileRank').textContent = d.rank || 'Bronze';
            document.getElementById('profileJoinDate').textContent = formatDate(d.joinDate);
            document.getElementById('profileAvatar').src = d.photo || `https://ui-avatars.com/api/?name=${encodeURIComponent(d.name)}&background=3b82f6&color=fff&size=120`;
            document.getElementById('profileReferralCode').textContent = d.referralCode;

            // Wallet
            document.getElementById('walletBalance').textContent = formatCurrency(d.balance);
            document.getElementById('walletTotalEarnings').textContent = formatCurrency(d.totalEarnings);
            document.getElementById('walletReferralEarnings').textContent = formatCurrency(d.referralEarnings);

            const walletCoinsEl = document.getElementById('walletCoins');
            if (walletCoinsEl) walletCoinsEl.textContent = Math.floor(d.balance || 0);
            const walletWithdrawableEl = document.getElementById('walletWithdrawable');
            if (walletWithdrawableEl) walletWithdrawableEl.textContent = formatCurrency(d.withdrawableBalance || 0);

            // Withdraw page live info
            const withdrawBalanceEl = document.getElementById('withdrawBalance');
            if (withdrawBalanceEl) withdrawBalanceEl.textContent = formatCurrency(d.withdrawableBalance || 0);
            const noticeCoinsEl = document.getElementById('noticeCoins');
            if (noticeCoinsEl) noticeCoinsEl.textContent = Math.floor(d.balance || 0);
            const noticeWithdrawableEl = document.getElementById('noticeWithdrawable');
            if (noticeWithdrawableEl) noticeWithdrawableEl.textContent = formatCurrency(d.withdrawableBalance || 0);
            const convertNoticeEl = document.getElementById('convertNotice');
            if (convertNoticeEl) {
                convertNoticeEl.style.display = (d.withdrawableBalance || 0) >= AppState.settings.minWithdraw ? 'none' : 'flex';
            }

            // Daily bonus button state
            const lastBonus = d.lastBonusClaim || 0;
            const now = Date.now();
            const canClaimBonus = (now - lastBonus) >= 24 * 60 * 60 * 1000;
            const bonusBtn = document.getElementById('dailyBonusBtn');
            if (bonusBtn) {
                bonusBtn.disabled = !canClaimBonus;
                bonusBtn.textContent = canClaimBonus ? '🎁 Claim Daily Bonus' : '✓ Claimed Today';
                bonusBtn.classList.toggle('claimed', !canClaimBonus);
            }

            // Check-in button state
            const lastCheckIn = d.lastCheckIn || 0;
            const canCheckIn = (now - lastCheckIn) >= 24 * 60 * 60 * 1000;
            const checkInBtn = document.getElementById('checkInBtn');
            if (checkInBtn) {
                checkInBtn.disabled = !canCheckIn;
                checkInBtn.textContent = canCheckIn ? `📅 Check In (Day ${(d.checkInStreak || 0) + 1})` : '✓ Checked In';
                checkInBtn.classList.toggle('claimed', !canCheckIn);
            }

            // Spin button
            const lastSpin = d.lastSpinTime || 0;
            const canSpin = (now - lastSpin) >= 24 * 60 * 60 * 1000;
            const spinBtn = document.getElementById('spinBtn');
            if (spinBtn) {
                spinBtn.disabled = !canSpin;
                spinBtn.textContent = canSpin ? '🎡 Spin the Wheel' : '✓ Come back tomorrow';
            }

            // Scratch button
            const lastScratch = d.lastScratchTime || 0;
            const canScratch = (now - lastScratch) >= 24 * 60 * 60 * 1000;
            const scratchBtn = document.getElementById('scratchBtn');
            if (scratchBtn) {
                scratchBtn.disabled = !canScratch;
                scratchBtn.textContent = canScratch ? '🃏 Scratch Card' : '✓ Come back tomorrow';
            }
        }

        // ========================
        // 9. AUTH SCREEN RENDERING
        // ========================
        function renderAuthScreen() {
            const container = document.getElementById('authScreen');
            container.innerHTML = `
                <div class="auth-bg"></div>
                <div class="auth-container">
                    <div class="auth-logo">
                        <div class="logo-icon">💰</div>
                        <h1 class="logo-text">Task & Earn <span>Pro</span></h1>
                        <p class="logo-tagline">Complete tasks. Earn money. Withdraw instantly.</p>
                    </div>

                    <div class="auth-tabs">
                        <button class="auth-tab active" data-tab="login" onclick="switchAuthTab('login')">Login</button>
                        <button class="auth-tab" data-tab="register" onclick="switchAuthTab('register')">Register</button>
                    </div>

                    <!-- Login Form -->
                    <form class="auth-form active" id="loginForm" onsubmit="event.preventDefault(); handleLogin(document.getElementById('loginEmail').value, document.getElementById('loginPassword').value);">
                        <div class="input-group">
                            <span class="input-icon">📧</span>
                            <input type="email" id="loginEmail" placeholder="Email address" required>
                        </div>
                        <div class="input-group">
                            <span class="input-icon">🔒</span>
                            <input type="password" id="loginPassword" placeholder="Password" required>
                        </div>
                        <button type="button" class="forgot-link" onclick="showForgotPassword()">Forgot password?</button>
                        <button type="submit" class="btn btn-primary btn-full">Sign In</button>
                    </form>

                    <!-- Register Form -->
                    <form class="auth-form" id="registerForm" onsubmit="event.preventDefault(); handleRegister(document.getElementById('regName').value, document.getElementById('regEmail').value, document.getElementById('regPassword').value, document.getElementById('regReferral').value);">
                        <div class="input-group">
                            <span class="input-icon">👤</span>
                            <input type="text" id="regName" placeholder="Full name" required>
                        </div>
                        <div class="input-group">
                            <span class="input-icon">📧</span>
                            <input type="email" id="regEmail" placeholder="Email address" required>
                        </div>
                        <div class="input-group">
                            <span class="input-icon">🔒</span>
                            <input type="password" id="regPassword" placeholder="Password (min 6 chars)" required minlength="6">
                        </div>
                        <div class="input-group">
                            <span class="input-icon">🎁</span>
                            <input type="text" id="regReferral" placeholder="Referral code (optional)">
                        </div>
                        <button type="submit" class="btn btn-primary btn-full">Create Account</button>
                    </form>

                    <div class="auth-divider">
                        <span>or continue with</span>
                    </div>

                    <button class="btn btn-google btn-full" onclick="handleGoogleLogin()">
                        <svg width="20" height="20" viewBox="0 0 24 24"><path fill="#4285F4" d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92a5.06 5.06 0 0 1-2.2 3.32v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.1z"/><path fill="#34A853" d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z"/><path fill="#FBBC05" d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z"/><path fill="#EA4335" d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z"/></svg>
                        Sign in with Google
                    </button>

                    <p class="auth-footer">By continuing, you agree to our <a onclick="navigateTo('terms')">Terms</a> & <a onclick="navigateTo('privacy')">Privacy Policy</a></p>
                </div>
            `;
        }

        function switchAuthTab(tab) {
            document.querySelectorAll('.auth-tab').forEach(t => t.classList.toggle('active', t.dataset.tab === tab));
            document.querySelectorAll('.auth-form').forEach(f => f.classList.toggle('active', f.id === `${tab}Form`));
        }

        function showForgotPassword() {
            openModal('Reset Password', `
                <div class="input-group">
                    <span class="input-icon">📧</span>
                    <input type="email" id="forgotEmail" placeholder="Enter your email address" required>
                </div>
                <p style="color: var(--text-muted); font-size: 0.85rem; margin-top: 8px;">We'll send you a link to reset your password.</p>
            `, `
                <button class="btn btn-secondary" onclick="closeModal()">Cancel</button>
                <button class="btn btn-primary" onclick="handleForgotPassword(document.getElementById('forgotEmail').value)">Send Reset Link</button>
            `);
        }

        // ========================
        // 10. APP SCREEN RENDERING
        // ========================
        function renderAppScreen() {
            const container = document.getElementById('appScreen');
            container.innerHTML = `
                <!-- Header -->
                <header class="app-header">
                    <button class="header-btn" onclick="toggleSidebar()">
                        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="3" y1="6" x2="21" y2="6"/><line x1="3" y1="12" x2="21" y2="12"/><line x1="3" y1="18" x2="21" y2="18"/></svg>
                    </button>
                    <h1 class="header-title" id="headerTitle">Dashboard</h1>
                    <div class="header-right">
                        <div class="header-balance" id="headerBalance">৳0.00</div>
                        <img class="header-avatar" id="headerAvatar" src="" alt="Profile" onclick="navigateTo('profile')">
                    </div>
                </header>

                <!-- Sidebar -->
                <div class="sidebar-overlay" id="sidebarOverlay" onclick="toggleSidebar()"></div>
                <nav class="sidebar" id="sidebar">
                    <div class="sidebar-header">
                        <div class="logo-icon-sm">💰</div>
                        <span>Task & Earn Pro</span>
                    </div>
                    <div class="sidebar-menu">
                        <a class="sidebar-item" onclick="navigateTo('home'); toggleSidebar();"><span>🏠</span> Home</a>
                        <a class="sidebar-item" onclick="navigateTo('earn'); toggleSidebar();"><span>💼</span> Earn Money</a>
                        <a class="sidebar-item" onclick="navigateTo('wallet'); toggleSidebar();"><span>👛</span> Wallet</a>
                        <a class="sidebar-item" onclick="navigateTo('withdraw'); toggleSidebar();"><span>💸</span> Withdraw</a>
                        <a class="sidebar-item" onclick="navigateTo('referrals'); toggleSidebar();"><span>👥</span> Referrals</a>
                        <a class="sidebar-item" onclick="navigateTo('notifications'); toggleSidebar();"><span>🔔</span> Notifications</a>
                        <a class="sidebar-item" onclick="navigateTo('profile'); toggleSidebar();"><span>👤</span> Profile</a>
                        <a class="sidebar-item" onclick="navigateTo('support'); toggleSidebar();"><span>🎧</span> Support</a>
                        <a class="sidebar-item" onclick="navigateTo('privacy'); toggleSidebar();"><span>🔒</span> Privacy Policy</a>
                        <a class="sidebar-item" onclick="navigateTo('terms'); toggleSidebar();"><span>📜</span> Terms & Conditions</a>
                        ${AppState.isAdmin ? '<a class="sidebar-item admin-item" onclick="navigateTo(\'admin\'); toggleSidebar();"><span>⚙️</span> Admin Panel</a>' : ''}
                    </div>
                    <div class="sidebar-footer">
                        <button class="btn btn-danger btn-full" onclick="handleLogout()">Sign Out</button>
                    </div>
                </nav>

                <!-- Main Content -->
                <main class="app-content" id="appContent">
                    <!-- Home Page -->
                    <div class="page active" id="page-home">
                        <!-- Announcement Banner -->
                        <div class="announcement-banner" id="announcementBanner" style="display:none;">
                            <span class="announcement-icon">📢</span>
                            <span class="announcement-text" id="announcementText"></span>
                        </div>

                        <!-- Banner Ad -->
                        <div class="ad-banner" id="bannerAd"></div>

                        <!-- Welcome Card -->
                        <div class="welcome-card">
                            <div class="welcome-left">
                                <img class="welcome-avatar" id="homeAvatar" src="" alt="">
                                <div>
                                    <h3 id="homeUserName">User</h3>
                                    <p id="homeUserEmail">user@email.com</p>
                                </div>
                            </div>
                            <div class="rank-badge">
                                <span class="rank-icon">🏆</span>
                                <span id="homeRank">Bronze</span>
                            </div>
                        </div>

                        <!-- Balance Card -->
                        <div class="balance-card">
                            <div class="balance-main">
                                <span class="balance-label">Total Balance</span>
                                <span class="balance-amount" id="homeBalance">৳0.00</span>
                            </div>
                            <div class="balance-stats">
                                <div class="balance-stat">
                                    <span class="stat-label">Total Earnings</span>
                                    <span class="stat-value" id="homeTotalEarnings">৳0.00</span>
                                </div>
                                <div class="balance-stat">
                                    <span class="stat-label">Referral Earnings</span>
                                    <span class="stat-value" id="homeReferralEarnings">৳0.00</span>
                                </div>
                                <div class="balance-stat">
                                    <span class="stat-label">Today's Earnings</span>
                                    <span class="stat-value" id="homeTodayEarnings">৳0.00</span>
                                </div>
                            </div>
                        </div>

                        <!-- Quick Actions -->
                        <div class="quick-actions">
                            <button class="quick-action" onclick="navigateTo('earn')">
                                <span class="quick-icon">💼</span>
                                <span>Earn</span>
                            </button>
                            <button class="quick-action" onclick="navigateTo('withdraw')">
                                <span class="quick-icon">💸</span>
                                <span>Withdraw</span>
                            </button>
                            <button class="quick-action" onclick="navigateTo('referrals')">
                                <span class="quick-icon">👥</span>
                                <span>Refer</span>
                            </button>
                            <button class="quick-action" onclick="navigateTo('wallet')">
                                <span class="quick-icon">👛</span>
                                <span>Wallet</span>
                            </button>
                        </div>

                        <!-- Daily Bonus -->
                        <div class="section-card">
                            <div class="section-header">
                                <h3>🎁 Daily Bonus</h3>
                            </div>
                            <button class="btn btn-primary btn-full" id="dailyBonusBtn" onclick="claimDailyBonus()">🎁 Claim Daily Bonus</button>
                        </div>

                        <!-- Daily Check-In -->
                        <div class="section-card">
                            <div class="section-header">
                                <h3>📅 Daily Check-In</h3>
                                <span class="streak-badge" id="checkInStreak">0 day streak</span>
                            </div>
                            <button class="btn btn-secondary btn-full" id="checkInBtn" onclick="claimDailyBonus()">📅 Check In (Day 1)</button>
                        </div>

                        <!-- Spin & Scratch -->
                        <div class="games-grid">
                            <div class="game-card">
                                <span class="game-icon">🎡</span>
                                <h4>Spin Wheel</h4>
                                <p>Win up to ৳50</p>
                                <button class="btn btn-primary btn-sm" id="spinBtn" onclick="spinWheel()">Spin Now</button>
                            </div>
                            <div class="game-card">
                                <span class="game-icon">🃏</span>
                                <h4>Scratch Card</h4>
                                <p>Win up to ৳30</p>
                                <button class="btn btn-primary btn-sm" id="scratchBtn" onclick="scratchCard()">Scratch Now</button>
                            </div>
                        </div>

                        <!-- Recent Transactions -->
                        <div class="section-card">
                            <div class="section-header">
                                <h3>📊 Recent Transactions</h3>
                                <button class="btn-link" onclick="navigateTo('wallet')">View All</button>
                            </div>
                            <div id="recentTransactions">
                                <p class="empty-state">No transactions yet</p>
                            </div>
                        </div>
                    </div>

                    <!-- Earn Page -->
                    <div class="page" id="page-earn">
                        <div class="section-card">
                            <div class="section-header">
                                <h3>💼 Available Tasks</h3>
                                <span class="badge" id="taskCount">0 tasks</span>
                            </div>
                            <div class="task-filters">
                                <button class="filter-btn active" data-filter="all" onclick="filterTasks('all')">All</button>
                                <button class="filter-btn" data-filter="ads" onclick="filterTasks('ads')">Watch Ads</button>
                                <button class="filter-btn" data-filter="visit" onclick="filterTasks('visit')">Visit</button>
                                <button class="filter-btn" data-filter="social" onclick="filterTasks('social')">Social</button>
                                <button class="filter-btn" data-filter="install" onclick="filterTasks('install')">Install</button>
                                <button class="filter-btn" data-filter="quiz" onclick="filterTasks('quiz')">Ptc</button>
                            </div>
                            <div id="tasksList" class="tasks-list">
                                <p class="empty-state">Loading tasks...</p>
                            </div>
                        </div>
                    </div>

                    <!-- Wallet Page -->
                    <div class="page" id="page-wallet">
                        <div class="wallet-card">
                            <span class="wallet-label">Available Balance</span>
                            <span class="wallet-amount" id="walletBalance">৳0.00</span>
                            <div class="wallet-stats">
                                <div class="wallet-stat">
                                    <span>Total Earnings</span>
                                    <span id="walletTotalEarnings">৳0.00</span>
                                </div>
                                <div class="wallet-stat">
                                    <span>Referral Earnings</span>
                                    <span id="walletReferralEarnings">৳0.00</span>
                                </div>
                                <div class="wallet-stat">
                                    <span>Coins</span>
                                    <span id="walletCoins">0</span>
                                </div>
                                <div class="wallet-stat">
                                    <span>Withdrawable</span>
                                    <span id="walletWithdrawable">৳0.00</span>
                                </div>
                            </div>
                        </div>

                        <div class="section-card">
                            <div class="section-header">
                                <h3>💳 Convert Coins</h3>
                            </div>
                            <div class="input-group">
                                <span class="input-icon">🪙</span>
                                <input type="number" id="convertCoins" placeholder="Enter coins to convert" min="1">
                            </div>
                            <p class="conversion-rate">10000 Coin = ৳${AppState.settings.coinRate}</p>
                            <button class="btn btn-primary btn-full" onclick="convertCoins()">Convert to Money</button>
                        </div>

                        <div class="section-card">
                            <div class="section-header">
                                <h3>📜 Transaction History</h3>
                            </div>
                            <div id="transactionHistory">
                                <p class="empty-state">No transactions yet</p>
                            </div>
                        </div>
                    </div>

                    <!-- Withdraw Page -->
                    <div class="page" id="page-withdraw">
                        <div class="withdraw-info">
                            <div class="info-item">
                                <span class="info-label">Minimum Withdraw</span>
                                <span class="info-value">৳${AppState.settings.minWithdraw}</span>
                            </div>
                            <div class="info-item">
                                <span class="info-label">Processing Time</span>
                                <span class="info-value">24-48 hours</span>
                            </div>
                            <div class="info-item">
                                <span class="info-label">Withdrawable Balance</span>
                                <span class="info-value" id="withdrawBalance">৳0.00</span>
                            </div>
                        </div>

                        <div class="convert-notice" id="convertNotice">
                            <span>⚠️</span>
                            <div>
                                <strong>Convert coins first!</strong>
                                <p>Coins: <b id="noticeCoins">0</b> &nbsp;|&nbsp; Withdrawable: <b id="noticeWithdrawable">৳0.00</b></p>
                                <button class="btn-link" onclick="navigateTo('wallet')">Convert Coins →</button>
                            </div>
                        </div>

                        <div class="section-card">
                            <div class="section-header">
                                <h3>💸 Withdraw Money</h3>
                            </div>
                            <div class="method-selector">
                                <button class="method-btn active" data-method="bkash" onclick="selectMethod('bkash')">
                                    <span>📱</span> bKash
                                </button>
                                <button class="method-btn" data-method="nagad" onclick="selectMethod('nagad')">
                                    <span>📱</span> Nagad
                                </button>
                                <button class="method-btn" data-method="rocket" onclick="selectMethod('rocket')">
                                    <span>📱</span> Rocket
                                </button>
                            </div>
                            <div class="input-group">
                                <span class="input-icon">💰</span>
                                <input type="number" id="withdrawAmount" placeholder="Amount (min ৳${AppState.settings.minWithdraw})" min="${AppState.settings.minWithdraw}">
                            </div>
                            <div class="input-group">
                                <span class="input-icon">📱</span>
                                <input type="tel" id="withdrawNumber" placeholder="Your mobile number">
                            </div>
                            <button class="btn btn-primary btn-full" onclick="submitWithdraw()">Submit Withdrawal Request</button>
                        </div>

                        <div class="section-card">
                            <div class="section-header">
                                <h3>📋 Withdrawal History</h3>
                            </div>
                            <div id="withdrawalHistory">
                                <p class="empty-state">No withdrawals yet</p>
                            </div>
                        </div>
                    </div>

                    <!-- Referrals Page -->
                    <div class="page" id="page-referrals">
                        <div class="referral-hero">
                            <span class="referral-icon">👥</span>
                            <h2>Invite & Earn</h2>
                            <p>Earn ৳${AppState.settings.referralBonus} for every friend who joins!</p>
                        </div>

                        <div class="section-card">
                            <div class="section-header">
                                <h3>Your Referral Code</h3>
                            </div>
                            <div class="referral-code-box">
                                <span id="referralCode">------</span>
                                <button class="copy-btn" onclick="copyReferralLink()">📋 Copy</button>
                            </div>
                        </div>

                        <div class="section-card">
                            <div class="section-header">
                                <h3>Referral Link</h3>
                            </div>
                            <div class="referral-link-box">
                                <input type="text" id="referralLink" readonly>
                                <button class="btn btn-primary btn-sm" onclick="shareReferralLink()">Share</button>
                            </div>
                        </div>

                        <div class="referral-stats">
                            <div class="stat-card">
                                <span class="stat-icon">👥</span>
                                <span class="stat-number" id="referralCount">0</span>
                                <span class="stat-label">Total Referrals</span>
                            </div>
                            <div class="stat-card">
                                <span class="stat-icon">💰</span>
                                <span class="stat-number" id="referralEarningsDisplay">৳0.00</span>
                                <span class="stat-label">Referral Earnings</span>
                            </div>
                        </div>

                        <div class="section-card" style="margin-top: 16px;">
                            <button class="btn btn-primary btn-full" onclick="claimReferralBonus()">🎁 Claim Referral Bonus</button>
                        </div>
                    </div>

                    <!-- Notifications Page -->
                    <div class="page" id="page-notifications">
                        <div class="section-card">
                            <div class="section-header">
                                <h3>🔔 Notifications</h3>
                            </div>
                            <div id="notificationsList">
                                <p class="empty-state">No notifications yet</p>
                            </div>
                        </div>
                    </div>

                    <!-- Profile Page -->
                    <div class="page" id="page-profile">
                        <div class="profile-header">
                            <img class="profile-avatar" id="profileAvatar" src="" alt="">
                            <h2 id="profileName">User</h2>
                            <p id="profileEmail">user@email.com</p>
                            <span class="profile-rank" id="profileRank">Bronze</span>
                        </div>

                        <div class="section-card">
                            <div class="profile-info">
                                <div class="info-row">
                                    <span>Member Since</span>
                                    <span id="profileJoinDate">N/A</span>
                                </div>
                                <div class="info-row">
                                    <span>Referral Code</span>
                                    <span id="profileReferralCode">------</span>
                                </div>
                                <div class="info-row">
                                    <span>Email Verified</span>
                                    <span id="profileVerified">No</span>
                                </div>
                            </div>
                        </div>

                        <button class="btn btn-danger btn-full" onclick="handleLogout()">Sign Out</button>
                    </div>

                    <!-- Support Page -->
                    <div class="page" id="page-support">
                        <div class="section-card">
                            <h3>🎧 Support Center</h3>
                            <p>Need help? We're here for you 24/7.</p>
                            <div class="support-options">
                                <a href="mailto:${AppState.settings.supportEmail}" class="support-item">
                                    <span>📧</span>
                                    <div>
                                        <strong>Email Support</strong>
                                        <p>${AppState.settings.supportEmail}</p>
                                    </div>
                                </a>
                                <a href="https://t.me/Sahalam_09" target="_blank" class="support-item">
                                    <span>✈️</span>
                                    <div>
                                        <strong>Telegram Support</strong>
                                        <p>@Sahalam_09</p>
                                    </div>
                                </a>
                            </div>
                        </div>
                        <div class="section-card">
                            <h3>❓ FAQ</h3>
                            <div class="faq-list">
                                <div class="faq-item">
                                    <strong>How do I withdraw money?</strong>
                                    <p>Go to the Withdraw page, select your method (bKash, Nagad, Rocket), enter amount and number, then submit.</p>
                                </div>
                                <div class="faq-item">
                                    <strong>When will I receive my payment?</strong>
                                    <p>Withdrawals are processed within 24-48 hours after admin approval.</p>
                                </div>
                                <div class="faq-item">
                                    <strong>How does referral work?</strong>
                                    <p>Share your referral link. When someone registers using your link, you earn ৳${AppState.settings.referralBonus} bonus.</p>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- Privacy Policy Page -->
                    <div class="page" id="page-privacy">
                        <div class="legal-content">
                            <h2>Privacy Policy</h2>
                            <p>Last updated: ${new Date().toLocaleDateString()}</p>
                            <h3>1. Information We Collect</h3>
                            <p>We collect your name, email address, and profile information when you register. We also collect usage data to improve our services.</p>
                            <h3>2. How We Use Your Information</h3>
                            <p>We use your information to provide our services, process withdrawals, and send important notifications.</p>
                            <h3>3. Data Security</h3>
                            <p>We use Firebase security rules and authentication to protect your data. Your password is never stored in plain text.</p>
                            <h3>4. Third-Party Services</h3>
                            <p>We use Firebase for authentication and database services. Please review Firebase's privacy policy for more information.</p>
                            <h3>5. Contact Us</h3>
                            <p>For privacy concerns, contact us at ${AppState.settings.supportEmail}</p>
                        </div>
                    </div>

                    <!-- Terms Page -->
                    <div class="page" id="page-terms">
                        <div class="legal-content">
                            <h2>Terms & Conditions</h2>
                            <p>Last updated: ${new Date().toLocaleDateString()}</p>
                            <h3>1. Acceptance of Terms</h3>
                            <p>By using Task & Earn Pro, you agree to these terms. If you do not agree, please do not use our services.</p>
                            <h3>2. User Responsibilities</h3>
                            <p>You must provide accurate information. Creating multiple accounts, using bots, or fraudulent activity will result in a ban.</p>
                            <h3>3. Earnings & Withdrawals</h3>
                            <p>Earnings are based on completing tasks. Minimum withdrawal is ৳${AppState.settings.minWithdraw}. Payments are processed within 24-48 hours.</p>
                            <h3>4. Account Termination</h3>
                            <p>We reserve the right to terminate accounts that violate our terms or engage in fraudulent activity.</p>
                            <h3>5. Changes to Terms</h3>
                            <p>We may update these terms at any time. Continued use of the app constitutes acceptance of new terms.</p>
                        </div>
                    </div>

                    <!-- Admin Page -->
                    <div class="page" id="page-admin">
                        <div class="admin-header">
                            <h2>⚙️ Admin Panel</h2>
                            <button class="btn btn-danger btn-sm" onclick="adminLogout()">Exit Admin</button>
                        </div>

                        <!-- Admin Dashboard Stats -->
                        <div class="admin-stats">
                            <div class="admin-stat-card">
                                <span class="admin-stat-icon">👥</span>
                                <span class="admin-stat-value" id="adminTotalUsers">0</span>
                                <span class="admin-stat-label">Total Users</span>
                            </div>
                            <div class="admin-stat-card">
                                <span class="admin-stat-icon">🟢</span>
                                <span class="admin-stat-value" id="adminActiveUsers">0</span>
                                <span class="admin-stat-label">Active Today</span>
                            </div>
                            <div class="admin-stat-card">
                                <span class="admin-stat-icon">💸</span>
                                <span class="admin-stat-value" id="adminTotalWithdrawals">৳0</span>
                                <span class="admin-stat-label">Total Withdrawn</span>
                            </div>
                            <div class="admin-stat-card">
                                <span class="admin-stat-icon">⏳</span>
                                <span class="admin-stat-value" id="adminPendingWithdrawals">0</span>
                                <span class="admin-stat-label">Pending</span>
                            </div>
                        </div>

                        <!-- Admin Tabs -->
                        <div class="admin-tabs">
                            <button class="admin-tab active" data-tab="withdrawals" onclick="switchAdminTab('withdrawals')">Withdrawals</button>
                            <button class="admin-tab" data-tab="users" onclick="switchAdminTab('users')">Users</button>
                            <button class="admin-tab" data-tab="tasks" onclick="switchAdminTab('tasks')">Tasks</button>
                            <button class="admin-tab" data-tab="notices" onclick="switchAdminTab('notices')">Notices</button>
                            <button class="admin-tab" data-tab="settings" onclick="switchAdminTab('settings')">Settings</button>
                        </div>

                        <!-- Admin Content -->
                        <div class="admin-content" id="adminContent">
                            <!-- Withdrawals Tab -->
                            <div class="admin-panel active" id="admin-withdrawals">
                                <div id="adminWithdrawalsList">
                                    <p class="empty-state">Loading...</p>
                                </div>
                            </div>

                            <!-- Users Tab -->
                            <div class="admin-panel" id="admin-users">
                                <div class="search-box">
                                    <input type="text" id="adminUserSearch" placeholder="Search users..." oninput="searchAdminUsers(this.value)">
                                </div>
                                <div id="adminUsersList">
                                    <p class="empty-state">Loading...</p>
                                </div>
                            </div>

                            <!-- Tasks Tab -->
                            <div class="admin-panel" id="admin-tasks">
                                <button class="btn btn-primary btn-full" onclick="showCreateTaskForm()">+ Create New Task</button>
                                <div id="adminTasksList">
                                    <p class="empty-state">Loading...</p>
                                </div>
                            </div>

                            <!-- Notices Tab -->
                            <div class="admin-panel" id="admin-notices">
                                <div class="input-group">
                                    <input type="text" id="noticeTitle" placeholder="Notice title">
                                </div>
                                <div class="input-group">
                                    <textarea id="noticeMessage" placeholder="Notice message" rows="3"></textarea>
                                </div>
                                <button class="btn btn-primary btn-full" onclick="adminSendNotification()">Send Notification</button>
                                <div id="adminNoticesList" style="margin-top: 16px;"></div>
                            </div>

                            <!-- Settings Tab -->
                            <div class="admin-panel" id="admin-settings">
                                <div class="input-group">
                                    <label>Minimum Withdraw (৳)</label>
                                    <input type="number" id="settingMinWithdraw" value="${AppState.settings.minWithdraw}">
                                </div>
                                <div class="input-group">
                                    <label>Coin Rate (1 coin = ৳)</label>
                                    <input type="number" id="settingCoinRate" value="${AppState.settings.coinRate}" step="0.01">
                                </div>
                                <div class="input-group">
                                    <label>Referral Bonus (৳)</label>
                                    <input type="number" id="settingReferralBonus" value="${AppState.settings.referralBonus}">
                                </div>
                                <div class="input-group">
                                    <label>Daily Bonus Base (৳)</label>
                                    <input type="number" id="settingDailyBonus" value="${AppState.settings.dailyBonusBase}">
                                </div>
                                <button class="btn btn-primary btn-full" onclick="adminUpdateSettings()">Save Settings</button>
                            </div>
                        </div>
                    </div>
                </main>

                <!-- Bottom Navigation -->
                <nav class="bottom-nav">
                    <button class="nav-item active" data-page="home" onclick="navigateTo('home')">
                        <span class="nav-icon">🏠</span>
                        <span class="nav-label">Home</span>
                    </button>
                    <button class="nav-item" data-page="earn" onclick="navigateTo('earn')">
                        <span class="nav-icon">💼</span>
                        <span class="nav-label">Earn</span>
                    </button>
                    <button class="nav-item" data-page="wallet" onclick="navigateTo('wallet')">
                        <span class="nav-icon">👛</span>
                        <span class="nav-label">Wallet</span>
                    </button>
                    <button class="nav-item" data-page="withdraw" onclick="navigateTo('withdraw')">
                        <span class="nav-icon">💸</span>
                        <span class="nav-label">Withdraw</span>
                    </button>
                    <button class="nav-item" data-page="referrals" onclick="navigateTo('referrals')">
                        <span class="nav-icon">👥</span>
                        <span class="nav-label">Refer</span>
                    </button>
                </nav>
            `;
        }

        // ========================
        // 11. PAGE DATA LOADERS
        // ========================
        function loadHomeData() {
            if (!AppState.user) return;
            const uid = AppState.user.uid;

            // Load recent transactions
            const txRef = firebaseFns.query(firebaseFns.ref(db, `transactions/${uid}`), firebaseFns.orderByChild('date'));
            firebaseFns.onValue(txRef, (snapshot) => {
                const container = document.getElementById('recentTransactions');
                if (!container) return;
                if (!snapshot.exists()) {
                    container.innerHTML = '<p class="empty-state">No transactions yet</p>';
                    return;
                }
                const transactions = [];
                snapshot.forEach(child => {
                    transactions.push({ id: child.key, ...child.val() });
                });
                transactions.sort((a, b) => b.date - a.date);
                const recent = transactions.slice(0, 5);
                container.innerHTML = recent.map(tx => `
                    <div class="transaction-item">
                        <div class="tx-icon">${tx.type === 'earning' ? '💰' : tx.type === 'withdrawal' ? '💸' : tx.type === 'bonus' ? '🎁' : '👥'}</div>
                        <div class="tx-details">
                            <span class="tx-desc">${tx.description || tx.type}</span>
                            <span class="tx-date">${formatDateTime(tx.date)}</span>
                        </div>
                        <span class="tx-amount ${tx.amount > 0 ? 'positive' : 'negative'}">${tx.amount > 0 ? '+' : ''}${formatCurrency(tx.amount)}</span>
                    </div>
                `).join('');
            });
        }

        function loadEarnTasks() {
            const tasksRef = firebaseFns.query(firebaseFns.ref(db, 'tasks'), firebaseFns.orderByChild('active'), firebaseFns.equalTo(true));
            firebaseFns.onValue(tasksRef, (snapshot) => {
                const container = document.getElementById('tasksList');
                const countEl = document.getElementById('taskCount');
                if (!container) return;
                if (!snapshot.exists()) {
                    container.innerHTML = '<p class="empty-state">No tasks available right now. Check back later!</p>';
                    if (countEl) countEl.textContent = '0 tasks';
                    return;
                }
                const tasks = [];
                snapshot.forEach(child => {
                    tasks.push({ id: child.key, ...child.val() });
                });
                AppState.tasks = tasks;
                if (countEl) countEl.textContent = `${tasks.length} tasks`;
                renderTasks(tasks, 'all');
            });
        }

        let currentTaskFilter = 'all';
        function filterTasks(filter) {
            currentTaskFilter = filter;
            document.querySelectorAll('.filter-btn').forEach(btn => {
                btn.classList.toggle('active', btn.dataset.filter === filter);
            });
            renderTasks(AppState.tasks, filter);
        }

        function renderTasks(tasks, filter) {
            const container = document.getElementById('tasksList');
            if (!container) return;
            let filtered = tasks;
            if (filter !== 'all') {
                filtered = tasks.filter(t => t.type === filter);
            }
            if (filtered.length === 0) {
                container.innerHTML = '<p class="empty-state">No tasks in this category</p>';
                return;
            }
            const completedTasks = AppState.userData?.completedTasks || {};
            container.innerHTML = filtered.map(task => {
                const isCompleted = completedTasks[task.id];
                return `
                    <div class="task-item ${isCompleted ? 'completed' : ''}">
                        <div class="task-icon">${getTaskIcon(task.type)}</div>
                        <div class="task-info">
                            <h4>${sanitizeInput(task.title)}</h4>
                            <p>${sanitizeInput(task.description || task.type)}</p>
                            <span class="task-reward">+${formatCurrency(task.reward)}</span>
                        </div>
                        <button class="btn ${isCompleted ? 'btn-success' : 'btn-primary'} btn-sm" 
                            onclick="completeTask('${task.id}', '${sanitizeInput(task.title)}', ${task.reward}, '${task.link || ''}')"
                            ${isCompleted ? 'disabled' : ''}>
                            ${isCompleted ? '✓ Done' : 'Start'}
                        </button>
                    </div>
                `;
            }).join('');
        }

        function getTaskIcon(type) {
            const icons = {
                ads: '📺', visit: '🌐', social: '📱', telegram: '✈️',
                install: '📲', ptc: '❓', survey: '📋', video: '🎬'
            };
            return icons[type] || '📌';
        }

        function loadWalletData() {
            if (!AppState.user) return;
            const uid = AppState.user.uid;
            const txRef = firebaseFns.query(firebaseFns.ref(db, `transactions/${uid}`), firebaseFns.orderByChild('date'));
            firebaseFns.onValue(txRef, (snapshot) => {
                const container = document.getElementById('transactionHistory');
                if (!container) return;
                if (!snapshot.exists()) {
                    container.innerHTML = '<p class="empty-state">No transactions yet</p>';
                    return;
                }
                const transactions = [];
                snapshot.forEach(child => {
                    transactions.push({ id: child.key, ...child.val() });
                });
                transactions.sort((a, b) => b.date - a.date);
                container.innerHTML = transactions.map(tx => `
                    <div class="transaction-item">
                        <div class="tx-icon">${tx.type === 'earning' ? '💰' : tx.type === 'withdrawal' ? '💸' : tx.type === 'bonus' ? '🎁' : '👥'}</div>
                        <div class="tx-details">
                            <span class="tx-desc">${tx.description || tx.type}</span>
                            <span class="tx-date">${formatDateTime(tx.date)}</span>
                        </div>
                        <span class="tx-amount ${tx.amount > 0 ? 'positive' : 'negative'}">${tx.amount > 0 ? '+' : ''}${formatCurrency(tx.amount)}</span>
                    </div>
                `).join('');
            });
        }

        function loadWithdrawPage() {
            if (!AppState.userData) return;
            document.getElementById('withdrawBalance').textContent = formatCurrency(AppState.userData.withdrawableBalance || 0);
            document.getElementById('withdrawAmount').min = AppState.settings.minWithdraw;
            document.getElementById('withdrawAmount').placeholder = `Amount (min ৳${AppState.settings.minWithdraw})`;

            const noticeCoins = document.getElementById('noticeCoins');
            const noticeWithdrawable = document.getElementById('noticeWithdrawable');
            const convertNotice = document.getElementById('convertNotice');
            if (noticeCoins) noticeCoins.textContent = Math.floor(AppState.userData.balance || 0);
            if (noticeWithdrawable) noticeWithdrawable.textContent = formatCurrency(AppState.userData.withdrawableBalance || 0);
            if (convertNotice) {
                convertNotice.style.display = (AppState.userData.withdrawableBalance || 0) >= AppState.settings.minWithdraw ? 'none' : 'flex';
            }

            const uid = AppState.user.uid;
            const wRef = firebaseFns.query(firebaseFns.ref(db, 'withdrawals'), firebaseFns.orderByChild('userId'), firebaseFns.equalTo(uid));
            firebaseFns.onValue(wRef, (snapshot) => {
                const container = document.getElementById('withdrawalHistory');
                if (!container) return;
                if (!snapshot.exists()) {
                    container.innerHTML = '<p class="empty-state">No withdrawals yet</p>';
                    return;
                }
                const withdrawals = [];
                snapshot.forEach(child => {
                    withdrawals.push({ id: child.key, ...child.val() });
                });
                withdrawals.sort((a, b) => b.date - a.date);
                container.innerHTML = withdrawals.map(w => `
                    <div class="withdrawal-item">
                        <div class="w-info">
                            <span class="w-method">${w.method.toUpperCase()}</span>
                            <span class="w-number">${w.number}</span>
                            <span class="w-date">${formatDateTime(w.date)}</span>
                        </div>
                        <div class="w-right">
                            <span class="w-amount">${formatCurrency(w.amount)}</span>
                            <span class="w-status status-${w.status}">${w.status}</span>
                        </div>
                    </div>
                `).join('');
            });
        }

        function loadReferralData() {
            if (!AppState.userData) return;
            const d = AppState.userData;
            document.getElementById('referralCode').textContent = d.referralCode;
            document.getElementById('referralCount').textContent = d.referralCount || 0;
            document.getElementById('referralEarningsDisplay').textContent = formatCurrency(d.referralEarnings);
            const refLink = `${window.location.origin}${window.location.pathname}?ref=${d.referralCode}`;
            document.getElementById('referralLink').value = refLink;
        }

        function loadNotifications() {
            const notifRef = firebaseFns.query(firebaseFns.ref(db, 'notifications'), firebaseFns.orderByChild('date'));
            firebaseFns.onValue(notifRef, (snapshot) => {
                const container = document.getElementById('notificationsList');
                if (!container) return;
                if (!snapshot.exists()) {
                    container.innerHTML = '<p class="empty-state">No notifications yet</p>';
                    return;
                }
                const notifications = [];
                snapshot.forEach(child => {
                    notifications.push({ id: child.key, ...child.val() });
                });
                notifications.sort((a, b) => b.date - a.date);
                container.innerHTML = notifications.slice(0, 50).map(n => `
                    <div class="notification-item ${n.read ? '' : 'unread'}">
                        <div class="notif-icon">${n.type === 'reward' ? '🎁' : n.type === 'announcement' ? '📢' : '🔔'}</div>
                        <div class="notif-content">
                            <h4>${sanitizeInput(n.title)}</h4>
                            <p>${sanitizeInput(n.message)}</p>
                            <span class="notif-date">${formatDateTime(n.date)}</span>
                        </div>
                    </div>
                `).join('');
            });
        }

        function loadProfileData() {
            if (!AppState.userData || !AppState.user) return;
            const d = AppState.userData;
            document.getElementById('profileVerified').textContent = AppState.user.emailVerified ? '✅ Yes' : '❌ No';
        }

        // ========================
        // 12. EARNING ACTIONS
        // ========================
        async function claimDailyBonus() {
            if (!AppState.user || !AppState.userData) return;
            const now = Date.now();
            const lastBonus = AppState.userData.lastBonusClaim || 0;

            if (now - lastBonus < 24 * 60 * 60 * 1000) {
                showToast('You already claimed your bonus today!', 'error');
                return;
            }

            showLoading('Claiming bonus...');
            try {
                const bonus = AppState.settings.dailyBonusBase + Math.floor(Math.random() * 20);
                const uid = AppState.user.uid;

                await firebaseFns.update(firebaseFns.ref(db, `users/${uid}`), {
                    balance: firebaseFns.increment(bonus),
                    totalEarnings: firebaseFns.increment(bonus),
                    todayEarnings: firebaseFns.increment(bonus),
                    lastBonusClaim: now
                });

                await firebaseFns.push(firebaseFns.ref(db, `transactions/${uid}`), {
                    type: 'bonus',
                    amount: bonus,
                    description: 'Daily bonus',
                    date: now,
                    status: 'completed'
                });

                await firebaseFns.push(firebaseFns.ref(db, 'notifications'), {
                    userId: uid,
                    title: 'Daily Bonus Claimed!',
                    message: `You earned ${formatCurrency(bonus)} from daily bonus.`,
                    type: 'reward',
                    date: now,
                    read: false
                });

                showToast(`You earned ${formatCurrency(bonus)}!`, 'success');
            } catch (error) {
                showToast('Failed to claim bonus', 'error');
            } finally {
                hideLoading();
            }
        }

        async function claimReferralBonus() {
            if (!AppState.user || !AppState.userData) return;

            if (AppState.userData.referralBonusClaimed) {
                showToast('You have already claimed your referral bonus!', 'error');
                return;
            }

            showLoading('Claiming referral bonus...');
            try {
                const uid = AppState.user.uid;
                const bonus = AppState.settings.referralBonus || 50;

                await firebaseFns.update(firebaseFns.ref(db, `users/${uid}`), {
                    balance: firebaseFns.increment(bonus),
                    totalEarnings: firebaseFns.increment(bonus),
                    referralBonusClaimed: true
                });

                await firebaseFns.push(firebaseFns.ref(db, `transactions/${uid}`), {
                    type: 'bonus',
                    amount: bonus,
                    description: 'Referral bonus claim',
                    date: Date.now(),
                    status: 'completed'
                });

                showToast(`You earned ${formatCurrency(bonus)}!`, 'success');
            } catch (error) {
                showToast('Failed to claim bonus', 'error');
            } finally {
                hideLoading();
            }
        }

        async function spinWheel() {
            if (!AppState.user || !AppState.userData) return;
            const now = Date.now();
            const lastSpin = AppState.userData.lastSpinTime || 0;

            if (now - lastSpin < 24 * 60 * 60 * 1000) {
                showToast('You already spun today! Come back tomorrow.', 'error');
                return;
            }

            showLoading('Spinning...');
            try {
                const reward = AppState.settings.spinRewardMin + Math.floor(Math.random() * (AppState.settings.spinRewardMax - AppState.settings.spinRewardMin + 1));
                const uid = AppState.user.uid;

                await firebaseFns.update(firebaseFns.ref(db, `users/${uid}`), {
                    balance: firebaseFns.increment(reward),
                    totalEarnings: firebaseFns.increment(reward),
                    todayEarnings: firebaseFns.increment(reward),
                    lastSpinTime: now,
                    spinCount: firebaseFns.increment(1)
                });

                await firebaseFns.push(firebaseFns.ref(db, `transactions/${uid}`), {
                    type: 'bonus',
                    amount: reward,
                    description: 'Spin wheel reward',
                    date: now,
                    status: 'completed'
                });

                showToast(`🎡 You won ${formatCurrency(reward)}!`, 'success');
            } catch (error) {
                showToast('Spin failed', 'error');
            } finally {
                hideLoading();
            }
        }

        async function scratchCard() {
            if (!AppState.user || !AppState.userData) return;
            const now = Date.now();
            const lastScratch = AppState.userData.lastScratchTime || 0;

            if (now - lastScratch < 24 * 60 * 60 * 1000) {
                showToast('You already scratched today! Come back tomorrow.', 'error');
                return;
            }

            showLoading('Scratching...');
            try {
                const reward = 5 + Math.floor(Math.random() * 26);
                const uid = AppState.user.uid;

                await firebaseFns.update(firebaseFns.ref(db, `users/${uid}`), {
                    balance: firebaseFns.increment(reward),
                    totalEarnings: firebaseFns.increment(reward),
                    todayEarnings: firebaseFns.increment(reward),
                    lastScratchTime: now,
                    scratchCount: firebaseFns.increment(1)
                });

                await firebaseFns.push(firebaseFns.ref(db, `transactions/${uid}`), {
                    type: 'bonus',
                    amount: reward,
                    description: 'Scratch card reward',
                    date: now,
                    status: 'completed'
                });

                showToast(`🃏 You won ${formatCurrency(reward)}!`, 'success');
            } catch (error) {
                showToast('Scratch failed', 'error');
            } finally {
                hideLoading();
            }
        }

        async function completeTask(taskId, taskTitle, reward, link) {
            if (!AppState.user || !AppState.userData) return;

            // Check if already completed
            if (AppState.userData.completedTasks && AppState.userData.completedTasks[taskId]) {
                showToast('You already completed this task!', 'error');
                return;
            }

            // Open link if provided
            if (link) {
                window.open(link, '_blank');
            }

            // Simulate ad watching for ad tasks
            if (link && link.includes('ad')) {
                await watchAd();
            }

            showLoading('Completing task...');
            try {
                const uid = AppState.user.uid;

                // Mark task as completed
                await firebaseFns.update(firebaseFns.ref(db, `users/${uid}/completedTasks`), {
                    [taskId]: true
                });

                // Add reward
                await firebaseFns.update(firebaseFns.ref(db, `users/${uid}`), {
                    balance: firebaseFns.increment(reward),
                    totalEarnings: firebaseFns.increment(reward),
                    todayEarnings: firebaseFns.increment(reward)
                });

                // Add transaction
                await firebaseFns.push(firebaseFns.ref(db, `transactions/${uid}`), {
                    type: 'earning',
                    amount: reward,
                    description: `Task: ${taskTitle}`,
                    date: Date.now(),
                    status: 'completed'
                });

                showToast(`✅ Task completed! Earned ${formatCurrency(reward)}`, 'success');
            } catch (error) {
                showToast('Failed to complete task', 'error');
            } finally {
                hideLoading();
            }
        }

        // ========================
        // 13. WITHDRAWAL
        // ========================
        let selectedMethod = 'bkash';

        function selectMethod(method) {
            selectedMethod = method;
            document.querySelectorAll('.method-btn').forEach(btn => {
                btn.classList.toggle('active', btn.dataset.method === method);
            });
        }

        async function submitWithdraw() {
            if (!AppState.user || !AppState.userData) return;

            const amount = parseFloat(document.getElementById('withdrawAmount').value);
            const number = document.getElementById('withdrawNumber').value.trim();

            const withdrawable = AppState.userData.withdrawableBalance || 0;

            // 🔒 Must convert coins to money first
            if (withdrawable <= 0) {
                showToast('আগে কয়েন থেকে টাকা Convert করুন, তারপর উইথড্র করুন!', 'error');
                navigateTo('wallet');
                return;
            }

            if (!amount || amount < AppState.settings.minWithdraw) {
                showToast(`Minimum withdrawal is ${formatCurrency(AppState.settings.minWithdraw)}`, 'error');
                return;
            }

            if (amount > withdrawable) {
                showToast('Insufficient withdrawable balance. Please convert more coins.', 'error');
                return;
            }

            if (!validatePhone(number)) {
                showToast('Please enter a valid Bangladeshi mobile number (01XXXXXXXXX)', 'error');
                return;
            }

            showLoading('Submitting request...');
            try {
                const uid = AppState.user.uid;

                await firebaseFns.push(firebaseFns.ref(db, 'withdrawals'), {
                    userId: uid,
                    userName: AppState.userData.name,
                    userEmail: AppState.userData.email,
                    method: selectedMethod,
                    amount: amount,
                    number: number,
                    status: 'pending',
                    date: Date.now()
                });

                // Deduct withdrawable balance
                await firebaseFns.update(firebaseFns.ref(db, `users/${uid}`), {
                    withdrawableBalance: firebaseFns.increment(-amount),
                    withdrawalCount: firebaseFns.increment(1)
                });

                // Add transaction
                await firebaseFns.push(firebaseFns.ref(db, `transactions/${uid}`), {
                    type: 'withdrawal',
                    amount: -amount,
                    description: `Withdrawal via ${selectedMethod.toUpperCase()}`,
                    date: Date.now(),
                    status: 'pending'
                });

                showToast('Withdrawal request submitted! Processing within 24-48 hours.', 'success');
                document.getElementById('withdrawAmount').value = '';
                document.getElementById('withdrawNumber').value = '';
            } catch (error) {
                showToast('Failed to submit withdrawal', 'error');
            } finally {
                hideLoading();
            }
        }

        // ========================
        // 14. COIN CONVERSION
        // ========================
        async function convertCoins() {
            if (!AppState.user || !AppState.userData) return;

            const coins = parseInt(document.getElementById('convertCoins').value);
            if (!coins || coins < 1) {
                showToast('Please enter a valid coin amount', 'error');
                return;
            }

            // Check if user has enough coins (assuming coins are separate from balance)
            // For this implementation, we'll treat balance as coins
            if (coins > AppState.userData.balance) {
                showToast('Insufficient coins', 'error');
                return;
            }

            const money = coins * AppState.settings.coinRate;
            showLoading('Converting...');
            try {
                const uid = AppState.user.uid;

                await firebaseFns.update(firebaseFns.ref(db, `users/${uid}`), {
                    balance: firebaseFns.increment(-coins),
                    withdrawableBalance: firebaseFns.increment(money)
                });

                await firebaseFns.push(firebaseFns.ref(db, `transactions/${uid}`), {
                    type: 'conversion',
                    amount: money,
                    description: `Converted ${coins} coins to ${formatCurrency(money)}`,
                    date: Date.now(),
                    status: 'completed'
                });

                showToast(`Converted ${coins} coins to ${formatCurrency(money)}. You can now withdraw!`, 'success');
                document.getElementById('convertCoins').value = '';
            } catch (error) {
                showToast('Conversion failed', 'error');
            } finally {
                hideLoading();
            }
        }

        // ========================
        // 15. REFERRAL (FIXED CLIPBOARD ISSUE)
        // ========================
        function copyReferralLink() {
            const linkInput = document.getElementById('referralLink');
            if (!linkInput) return;
            
            const link = linkInput.value;

            // Check if clipboard API is supported and we are in a secure context (HTTPS/localhost)
            if (navigator.clipboard && window.isSecureContext) {
                navigator.clipboard.writeText(link).then(() => {
                    showToast('Referral link copied!', 'success');
                }).catch((err) => {
                    console.warn('Clipboard API failed, trying fallback...', err);
                    fallbackCopyLink(linkInput);
                });
            } else {
                // Fallback for non-secure contexts or unsupported browsers
                fallbackCopyLink(linkInput);
            }
        }

        function fallbackCopyLink(inputElement) {
            try {
                inputElement.select();
                inputElement.setSelectionRange(0, 99999); // For mobile devices
                const successful = document.execCommand('copy');
                if (successful) {
                    showToast('Referral link copied!', 'success');
                } else {
                    showToast('Failed to copy link. Please copy manually.', 'error');
                }
            } catch (err) {
                console.error('Fallback copy failed: ', err);
                showToast('Failed to copy link. Please copy manually.', 'error');
            } finally {
                // Deselect the input
                if (window.getSelection) {
                    window.getSelection().removeAllRanges();
                }
            }
        }

        function shareReferralLink() {
            const link = document.getElementById('referralLink').value;
            if (navigator.share) {
                navigator.share({
                    title: 'Join Task & Earn Pro',
                    text: 'Earn money by completing simple tasks! Use my referral link:',
                    url: link
                });
            } else {
                copyReferralLink();
            }
        }

        // ========================
        // 16. ADMIN FUNCTIONS
        // ========================
        async function adminLogin(email, password) {
            showLoading('Admin login...');
            try {
                await firebaseFns.signInWithEmailAndPassword(auth, email, password);
                AppState.isAdmin = true;
                showToast('Admin access granted', 'success');
                navigateTo('admin');
            } catch (error) {
                showToast('Invalid admin credentials', 'error');
            } finally {
                hideLoading();
            }
        }

        function adminLogout() {
            AppState.isAdmin = false;
            navigateTo('home');
            showToast('Exited admin panel', 'info');
        }

        function switchAdminTab(tab) {
            document.querySelectorAll('.admin-tab').forEach(t => {
                t.classList.toggle('active', t.dataset.tab === tab);
            });
            document.querySelectorAll('.admin-panel').forEach(p => {
                p.classList.toggle('active', p.id === `admin-${tab}`);
            });
            if (tab === 'withdrawals') loadAdminWithdrawals();
            if (tab === 'users') loadAdminUsers();
            if (tab === 'tasks') loadAdminTasks();
            if (tab === 'notices') loadAdminNotices();
        }

        function loadAdminDashboard() {
            // Load stats
            firebaseFns.onValue(firebaseFns.ref(db, 'users'), (snapshot) => {
                const users = snapshot.val() || {};
                const userCount = Object.keys(users).length;
                const today = new Date().setHours(0, 0, 0, 0);
                const activeToday = Object.values(users).filter(u => (u.lastLogin || 0) >= today).length;
                document.getElementById('adminTotalUsers').textContent = userCount;
                document.getElementById('adminActiveUsers').textContent = activeToday;
            });

            firebaseFns.onValue(firebaseFns.ref(db, 'withdrawals'), (snapshot) => {
                const withdrawals = snapshot.val() || {};
                const wList = Object.values(withdrawals);
                const total = wList.filter(w => w.status === 'approved').reduce((sum, w) => sum + (w.amount || 0), 0);
                const pending = wList.filter(w => w.status === 'pending').length;
                document.getElementById('adminTotalWithdrawals').textContent = formatCurrency(total);
                document.getElementById('adminPendingWithdrawals').textContent = pending;
            });

            loadAdminWithdrawals();
        }

        function loadAdminWithdrawals() {
            const wRef = firebaseFns.query(firebaseFns.ref(db, 'withdrawals'), firebaseFns.orderByChild('date'));
            firebaseFns.onValue(wRef, (snapshot) => {
                const container = document.getElementById('adminWithdrawalsList');
                if (!container) return;
                if (!snapshot.exists()) {
                    container.innerHTML = '<p class="empty-state">No withdrawal requests</p>';
                    return;
                }
                const withdrawals = [];
                snapshot.forEach(child => {
                    withdrawals.push({ id: child.key, ...child.val() });
                });
                withdrawals.sort((a, b) => b.date - a.date);
                container.innerHTML = withdrawals.map(w => `
                    <div class="admin-withdrawal-item">
                        <div class="admin-w-info">
                            <span class="w-user">${sanitizeInput(w.userName || w.userEmail)}</span>
                            <span class="w-method">${w.method.toUpperCase()}</span>
                            <span class="w-number">${w.number}</span>
                            <span class="w-date">${formatDateTime(w.date)}</span>
                        </div>
                        <div class="admin-w-right">
                            <span class="w-amount">${formatCurrency(w.amount)}</span>
                            <span class="w-status status-${w.status}">${w.status}</span>
                            ${w.status === 'pending' ? `
                                <div class="admin-actions">
                                    <button class="btn btn-success btn-xs" onclick="adminApproveWithdraw('${w.id}')">✓</button>
                                    <button class="btn btn-danger btn-xs" onclick="adminRejectWithdraw('${w.id}')">✕</button>
                                </div>
                            ` : ''}
                        </div>
                    </div>
                `).join('');
            });
        }

        async function adminApproveWithdraw(withdrawId) {
            showLoading('Approving...');
            try {
                const wRef = firebaseFns.ref(db, `withdrawals/${withdrawId}`);
                const snapshot = await firebaseFns.get(wRef);
                if (!snapshot.exists()) { showToast('Withdrawal not found', 'error'); return; }
                const w = snapshot.val();

                await firebaseFns.update(wRef, { status: 'approved', approvedAt: Date.now() });

                // Add notification
                await firebaseFns.push(firebaseFns.ref(db, 'notifications'), {
                    userId: w.userId,
                    title: 'Withdrawal Approved!',
                    message: `Your withdrawal of ${formatCurrency(w.amount)} via ${w.method.toUpperCase()} has been approved.`,
                    type: 'reward',
                    date: Date.now(),
                    read: false
                });

                showToast('Withdrawal approved', 'success');
                loadAdminWithdrawals();
            } catch (error) {
                showToast('Failed to approve', 'error');
            } finally {
                hideLoading();
            }
        }

        async function adminRejectWithdraw(withdrawId) {
            showLoading('Rejecting...');
            try {
                const wRef = firebaseFns.ref(db, `withdrawals/${withdrawId}`);
                const snapshot = await firebaseFns.get(wRef);
                if (!snapshot.exists()) { showToast('Withdrawal not found', 'error'); return; }
                const w = snapshot.val();

                await firebaseFns.update(wRef, { status: 'rejected', rejectedAt: Date.now() });

                // Refund withdrawable balance
                await firebaseFns.update(firebaseFns.ref(db, `users/${w.userId}`), {
                    withdrawableBalance: firebaseFns.increment(w.amount)
                });

                // Add notification
                await firebaseFns.push(firebaseFns.ref(db, 'notifications'), {
                    userId: w.userId,
                    title: 'Withdrawal Rejected',
                    message: `Your withdrawal of ${formatCurrency(w.amount)} was rejected. Amount refunded to your withdrawable balance.`,
                    type: 'announcement',
                    date: Date.now(),
                    read: false
                });

                showToast('Withdrawal rejected and refunded', 'info');
                loadAdminWithdrawals();
            } catch (error) {
                showToast('Failed to reject', 'error');
            } finally {
                hideLoading();
            }
        }

        function loadAdminUsers() {
            firebaseFns.onValue(firebaseFns.ref(db, 'users'), (snapshot) => {
                const container = document.getElementById('adminUsersList');
                if (!container) return;
                if (!snapshot.exists()) {
                    container.innerHTML = '<p class="empty-state">No users</p>';
                    return;
                }
                const users = [];
                snapshot.forEach(child => {
                    users.push({ id: child.key, ...child.val() });
                });
                users.sort((a, b) => (b.joinDate || 0) - (a.joinDate || 0));
                container.innerHTML = users.slice(0, 100).map(u => `
                    <div class="admin-user-item">
                        <div class="admin-user-info">
                            <span class="user-name">${sanitizeInput(u.name)} ${u.banned ? '🚫' : ''}</span>
                            <span class="user-email">${sanitizeInput(u.email)}</span>
                            <span class="user-balance">${formatCurrency(u.balance)}</span>
                        </div>
                        <div class="admin-user-actions">
                            <button class="btn btn-xs ${u.banned ? 'btn-success' : 'btn-danger'}" onclick="adminBanUser('${u.id}', ${!u.banned})">${u.banned ? 'Unban' : 'Ban'}</button>
                            <button class="btn btn-xs btn-primary" onclick="adminAddCoins('${u.id}')">+Coins</button>
                            <button class="btn btn-xs btn-secondary" onclick="adminRemoveCoins('${u.id}')">-Coins</button>
                        </div>
                    </div>
                `).join('');
            });
        }

        function searchAdminUsers(query) {
            // Simple search implementation
            const items = document.querySelectorAll('.admin-user-item');
            items.forEach(item => {
                const text = item.textContent.toLowerCase();
                item.style.display = text.includes(query.toLowerCase()) ? 'flex' : 'none';
            });
        }

        async function adminBanUser(uid, ban) {
            showLoading(ban ? 'Banning...' : 'Unbanning...');
            try {
                await firebaseFns.update(firebaseFns.ref(db, `users/${uid}`), { banned: ban });
                showToast(ban ? 'User banned' : 'User unbanned', 'success');
                loadAdminUsers();
            } catch (error) {
                showToast('Action failed', 'error');
            } finally {
                hideLoading();
            }
        }

        async function adminAddCoins(uid) {
            const amount = prompt('Enter amount to add:');
            if (!amount || isNaN(amount) || amount <= 0) return;
            showLoading('Adding coins...');
            try {
                await firebaseFns.update(firebaseFns.ref(db, `users/${uid}`), {
                    balance: firebaseFns.increment(parseFloat(amount)),
                    totalEarnings: firebaseFns.increment(parseFloat(amount))
                });
                await firebaseFns.push(firebaseFns.ref(db, `transactions/${uid}`), {
                    type: 'admin_credit',
                    amount: parseFloat(amount),
                    description: 'Admin credit',
                    date: Date.now(),
                    status: 'completed'
                });
                showToast(`Added ${formatCurrency(amount)}`, 'success');
                loadAdminUsers();
            } catch (error) {
                showToast('Failed', 'error');
            } finally {
                hideLoading();
            }
        }

        async function adminRemoveCoins(uid) {
            const amount = prompt('Enter amount to remove:');
            if (!amount || isNaN(amount) || amount <= 0) return;
            showLoading('Removing coins...');
            try {
                await firebaseFns.update(firebaseFns.ref(db, `users/${uid}`), {
                    balance: firebaseFns.increment(-parseFloat(amount)),
                    totalEarnings: firebaseFns.increment(-parseFloat(amount))
                });
                await firebaseFns.push(firebaseFns.ref(db, `transactions/${uid}`), {
                    type: 'admin_debit',
                    amount: -parseFloat(amount),
                    description: 'Admin debit',
                    date: Date.now(),
                    status: 'completed'
                });
                showToast(`Removed ${formatCurrency(amount)}`, 'success');
                loadAdminUsers();
            } catch (error) {
                showToast('Failed', 'error');
            } finally {
                hideLoading();
            }
        }

        function loadAdminTasks() {
            firebaseFns.onValue(firebaseFns.ref(db, 'tasks'), (snapshot) => {
                const container = document.getElementById('adminTasksList');
                if (!container) return;
                if (!snapshot.exists()) {
                    container.innerHTML = '<p class="empty-state">No tasks created</p>';
                    return;
                }
                const tasks = [];
                snapshot.forEach(child => {
                    tasks.push({ id: child.key, ...child.val() });
                });
                container.innerHTML = tasks.map(t => `
                    <div class="admin-task-item">
                        <div class="task-info">
                            <h4>${sanitizeInput(t.title)}</h4>
                            <p>${t.type} • ${formatCurrency(t.reward)} • ${t.active ? '✅ Active' : '❌ Inactive'}</p>
                        </div>
                        <div class="admin-task-actions">
                            <button class="btn btn-xs ${t.active ? 'btn-warning' : 'btn-success'}" onclick="toggleTaskActive('${t.id}', ${!t.active})">${t.active ? 'Deactivate' : 'Activate'}</button>
                            <button class="btn btn-xs btn-danger" onclick="adminDeleteTask('${t.id}')">Delete</button>
                        </div>
                    </div>
                `).join('');
            });
        }

        function showCreateTaskForm() {
            openModal('Create Task', `
                <div class="input-group">
                    <label>Task Title</label>
                    <input type="text" id="taskTitle" placeholder="e.g. Watch a video">
                </div>
                <div class="input-group">
                    <label>Reward (৳)</label>
                    <input type="number" id="taskReward" placeholder="e.g. 10" min="1">
                </div>
                <div class="input-group">
                    <label>Task Link</label>
                    <input type="url" id="taskLink" placeholder="https://...">
                </div>
                <div class="input-group">
                    <label>Task Type</label>
                    <select id="taskType">
                        <option value="ads">Watch Ads</option>
                        <option value="visit">Visit Website</option>
                        <option value="social">Social Media</option>
                        <option value="telegram">Telegram Join</option>
                        <option value="install">App Install</option>
                        <option value="quiz">ptc</option>
                    </select>
                </div>
            `, `
                <button class="btn btn-secondary" onclick="closeModal()">Cancel</button>
                <button class="btn btn-primary" onclick="adminCreateTask()">Create Task</button>
            `);
        }

        async function adminCreateTask() {
            const title = document.getElementById('taskTitle').value.trim();
            const reward = parseFloat(document.getElementById('taskReward').value);
            const link = document.getElementById('taskLink').value.trim();
            const type = document.getElementById('taskType').value;

            if (!title || !reward || reward <= 0) {
                showToast('Please fill all fields correctly', 'error');
                return;
            }

            showLoading('Creating task...');
            try {
                await firebaseFns.push(firebaseFns.ref(db, 'tasks'), {
                    title,
                    description: `${type} task`,
                    reward,
                    link,
                    type,
                    active: true,
                    createdAt: Date.now()
                });
                showToast('Task created!', 'success');
                closeModal();
                loadAdminTasks();
            } catch (error) {
                showToast('Failed to create task', 'error');
            } finally {
                hideLoading();
            }
        }

        async function adminDeleteTask(taskId) {
            if (!confirm('Delete this task?')) return;
            showLoading('Deleting...');
            try {
                await firebaseFns.remove(firebaseFns.ref(db, `tasks/${taskId}`));
                showToast('Task deleted', 'success');
                loadAdminTasks();
            } catch (error) {
                showToast('Failed to delete', 'error');
            } finally {
                hideLoading();
            }
        }

        async function toggleTaskActive(taskId, active) {
            showLoading('Updating...');
            try {
                await firebaseFns.update(firebaseFns.ref(db, `tasks/${taskId}`), { active });
                loadAdminTasks();
            } catch (error) {
                showToast('Failed', 'error');
            } finally {
                hideLoading();
            }
        }

        function loadAdminNotices() {
            firebaseFns.onValue(firebaseFns.ref(db, 'notifications'), (snapshot) => {
                const container = document.getElementById('adminNoticesList');
                if (!container) return;
                if (!snapshot.exists()) {
                    container.innerHTML = '<p class="empty-state">No notices sent</p>';
                    return;
                }
                const notifications = [];
                snapshot.forEach(child => {
                    notifications.push({ id: child.key, ...child.val() });
                });
                notifications.sort((a, b) => b.date - a.date);
                container.innerHTML = notifications.slice(0, 20).map(n => `
                    <div class="admin-notice-item">
                        <div>
                            <strong>${sanitizeInput(n.title)}</strong>
                            <p>${sanitizeInput(n.message)}</p>
                            <span class="notif-date">${formatDateTime(n.date)}</span>
                        </div>
                    </div>
                `).join('');
            });
        }

        async function adminSendNotification() {
            const title = document.getElementById('noticeTitle').value.trim();
            const message = document.getElementById('noticeMessage').value.trim();
            if (!title || !message) { showToast('Please fill all fields', 'error'); return; }

            showLoading('Sending...');
            try {
                await firebaseFns.push(firebaseFns.ref(db, 'notifications'), {
                    title,
                    message,
                    type: 'announcement',
                    date: Date.now(),
                    read: false,
                    userId: 'all'
                });
                // Also set as announcement banner
                await firebaseFns.set(firebaseFns.ref(db, 'settings/announcement'), {
                    title,
                    message,
                    active: true,
                    date: Date.now()
                });
                showToast('Notification sent!', 'success');
                document.getElementById('noticeTitle').value = '';
                document.getElementById('noticeMessage').value = '';
                loadAdminNotices();
            } catch (error) {
                showToast('Failed to send', 'error');
            } finally {
                hideLoading();
            }
        }

        async function adminUpdateSettings() {
            const minWithdraw = parseFloat(document.getElementById('settingMinWithdraw').value);
            const coinRate = parseFloat(document.getElementById('settingCoinRate').value);
            const referralBonus = parseFloat(document.getElementById('settingReferralBonus').value);
            const dailyBonusBase = parseFloat(document.getElementById('settingDailyBonus').value);

            showLoading('Saving settings...');
            try {
                await firebaseFns.update(firebaseFns.ref(db, 'settings'), {
                    minWithdraw,
                    coinRate,
                    referralBonus,
                    dailyBonusBase
                });
                showToast('Settings saved!', 'success');
            } catch (error) {
                showToast('Failed to save', 'error');
            } finally {
                hideLoading();
            }
        }

        // ========================
        // 17. AD INTEGRATION
        // ========================
        function loadBannerAd() {
            const container = document.getElementById('bannerAd');
            if (!container || !AppState.ads.banner) return;
            container.innerHTML = AppState.ads.banner;
        }

        function watchAd() {
            return new Promise((resolve) => {
                if (AppState.ads.rewarded) {
                    // Execute rewarded ad script
                    try {
                        // For Monetag: show_XXXXXXX().then(...)
                        // For Adsterra: similar pattern
                        const script = document.createElement('script');
                        script.innerHTML = AppState.ads.rewarded;
                        document.body.appendChild(script);
                        setTimeout(() => {
                            resolve(true);
                        }, 3000);
                    } catch (e) {
                        resolve(true);
                    }
                } else {
                    // Simulate ad watching
                    setTimeout(() => resolve(true), 2000);
                }
            });
        }

        // ========================
        // 18. ANNOUNCEMENT BANNER
        // ========================
        function loadAnnouncement() {
            firebaseFns.onValue(firebaseFns.ref(db, 'settings/announcement'), (snapshot) => {
                const banner = document.getElementById('announcementBanner');
                const text = document.getElementById('announcementText');
                if (!banner || !text) return;
                if (snapshot.exists() && snapshot.val().active) {
                    text.textContent = snapshot.val().message;
                    banner.style.display = 'flex';
                } else {
                    banner.style.display = 'none';
                }
            });
        }

        // ========================
        // 19. PWA SUPPORT
        // ========================
        function setupPWA() {
            // Generate manifest dynamically
            const manifest = {
                name: 'Task & Earn Pro',
                short_name: 'TaskEarn',
                description: 'Complete tasks and earn money',
                start_url: window.location.pathname,
                display: 'standalone',
                background_color: '#0a0e27',
                theme_color: '#3b82f6',
                icons: [
                    {
                        src: 'data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><rect fill="%233b82f6" width="100" height="100" rx="20"/><text x="50" y="65" font-size="50" text-anchor="middle" fill="white">💰</text></svg>',
                        sizes: '192x192',
                        type: 'image/svg+xml'
                    }
                ]
            };

            const manifestBlob = new Blob([JSON.stringify(manifest)], { type: 'application/json' });
            const manifestUrl = URL.createObjectURL(manifestBlob);

            // Update manifest link
            const manifestLink = document.querySelector('link[rel="manifest"]');
            if (manifestLink) manifestLink.href = manifestUrl;

            // Register service worker
            if ('serviceWorker' in navigator) {
                const swCode = `
                    const CACHE_NAME = 'task-earn-v1';
                    self.addEventListener('install', (e) => {
                        e.waitUntil(caches.open(CACHE_NAME).then(cache => cache.addAll(['/'])));
                        self.skipWaiting();
                    });
                    self.addEventListener('activate', (e) => {
                        e.waitUntil(clients.claim());
                    });
                    self.addEventListener('fetch', (e) => {
                        e.respondWith(
                            caches.match(e.request).then(response => response || fetch(e.request).then(res => {
                                const clone = res.clone();
                                caches.open(CACHE_NAME).then(cache => cache.put(e.request, clone));
                                return res;
                            }))
                        );
                    });
                `;
                const swBlob = new Blob([swCode], { type: 'application/javascript' });
                const swUrl = URL.createObjectURL(swBlob);
                navigator.serviceWorker.register(swUrl).catch(() => {});
            }
        }

        // ========================
        // 20. INITIALIZATION
        // ========================
        document.addEventListener('DOMContentLoaded', () => {
            renderAuthScreen();
            renderAppScreen();
            setupPWA();
            loadAnnouncement();

            // Check for referral code in URL
            const urlParams = new URLSearchParams(window.location.search);
            const refCode = urlParams.get('ref');
            if (refCode) {
                // Store for registration
                sessionStorage.setItem('referralCode', refCode);
            }

            // Pre-fill referral code if present
            setTimeout(() => {
                const regReferral = document.getElementById('regReferral');
                if (regReferral && sessionStorage.getItem('referralCode')) {
                    regReferral.value = sessionStorage.getItem('referralCode');
                }
            }, 500);
        });

        // Expose additional functions
        window.convertCoins = convertCoins;
        window.selectMethod = selectMethod;
        window.filterTasks = filterTasks;
        window.switchAuthTab = switchAuthTab;
        window.showForgotPassword = showForgotPassword;
        window.switchAdminTab = switchAdminTab;
        window.loadAdminDashboard = loadAdminDashboard;
        window.searchAdminUsers = searchAdminUsers;
        window.showCreateTaskForm = showCreateTaskForm;
        window.toggleTaskActive = toggleTaskActive;

        // Admin login helper
        window.showAdminLogin = function() {
            openModal('Admin Login', `
                <div class="input-group">
                    <span class="input-icon">📧</span>
                    <input type="email" id="adminEmail" placeholder="Admin email">
                </div>
                <div class="input-group">
                    <span class="input-icon">🔒</span>
                    <input type="password" id="adminPassword" placeholder="Admin password">
                </div>
            `, `
                <button class="btn btn-secondary" onclick="closeModal()">Cancel</button>
                <button class="btn btn-primary" onclick="adminLogin(document.getElementById('adminEmail').value, document.getElementById('adminPassword').value)">Login</button>
            `);
        };
    </script>

    <style>
        /* ========================
           CSS RESET & VARIABLES
           ======================== */
        *, *::before, *::after {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            -webkit-tap-highlight-color: transparent;
        }

        :root {
            --bg-primary: #0a0e27;
            --bg-secondary: #111638;
            --bg-card: #1a1f4a;
            --bg-hover: #232a5c;
            --primary: #3b82f6;
            --primary-dark: #2563eb;
            --primary-light: #60a5fa;
            --accent: #8b5cf6;
            --success: #10b981;
            --warning: #f59e0b;
            --danger: #ef4444;
            --text-primary: #ffffff;
            --text-secondary: #94a3b8;
            --text-muted: #64748b;
            --border: rgba(255, 255, 255, 0.08);
            --gradient-1: linear-gradient(135deg, #0a0e27 0%, #1a1f4a 100%);
            --gradient-2: linear-gradient(135deg, #3b82f6 0%, #8b5cf6 100%);
            --gradient-3: linear-gradient(135deg, #1e3a8a 0%, #3b82f6 100%);
            --shadow-sm: 0 2px 8px rgba(0, 0, 0, 0.2);
            --shadow-md: 0 4px 20px rgba(0, 0, 0, 0.3);
            --shadow-lg: 0 8px 40px rgba(0, 0, 0, 0.4);
            --shadow-glow: 0 0 30px rgba(59, 130, 246, 0.3);
            --radius-sm: 8px;
            --radius-md: 14px;
            --radius-lg: 20px;
            --radius-xl: 28px;
            --header-height: 60px;
            --nav-height: 65px;
            --safe-bottom: env(safe-area-inset-bottom, 0px);
        }

        html {
            font-size: 16px;
            scroll-behavior: smooth;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
            background: var(--bg-primary);
            color: var(--text-primary);
            min-height: 100vh;
            overflow-x: hidden;
            -webkit-font-smoothing: antialiased;
            -moz-osx-font-smoothing: grayscale;
        }

        /* ========================
           ANIMATIONS
           ======================== */
        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }

        @keyframes slideUp {
            from { opacity: 0; transform: translateY(30px); }
            to { opacity: 1; transform: translateY(0); }
        }

        @keyframes slideInRight {
            from { opacity: 0; transform: translateX(100px); }
            to { opacity: 1; transform: translateX(0); }
        }

        @keyframes slideOutRight {
            from { opacity: 1; transform: translateX(0); }
            to { opacity: 0; transform: translateX(100px); }
        }

        @keyframes pulse {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.05); }
        }

        @keyframes shimmer {
            0% { background-position: -200% 0; }
            100% { background-position: 200% 0; }
        }

        @keyframes spin {
            from { transform: rotate(0deg); }
            to { transform: rotate(360deg); }
        }

        @keyframes bounceIn {
            0% { transform: scale(0.3); opacity: 0; }
            50% { transform: scale(1.05); }
            70% { transform: scale(0.9); }
            100% { transform: scale(1); opacity: 1; }
        }

        /* ========================
           TOAST NOTIFICATIONS
           ======================== */
        #toastContainer {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 10000;
            display: flex;
            flex-direction: column;
            gap: 10px;
            max-width: 320px;
        }

        .toast {
            display: flex;
            align-items: center;
            gap: 10px;
            padding: 14px 18px;
            border-radius: var(--radius-md);
            background: var(--bg-card);
            border: 1px solid var(--border);
            box-shadow: var(--shadow-lg);
            animation: slideInRight 0.3s ease;
            font-size: 0.9rem;
        }

        .toast-success { border-left: 4px solid var(--success); }
        .toast-error { border-left: 4px solid var(--danger); }
        .toast-info { border-left: 4px solid var(--primary); }

        .toast-icon {
            width: 24px;
            height: 24px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 0.8rem;
            font-weight: bold;
            flex-shrink: 0;
        }

        .toast-success .toast-icon { background: rgba(16, 185, 129, 0.2); color: var(--success); }
        .toast-error .toast-icon { background: rgba(239, 68, 68, 0.2); color: var(--danger); }
        .toast-info .toast-icon { background: rgba(59, 130, 246, 0.2); color: var(--primary); }

        /* ========================
           LOADING OVERLAY
           ======================== */
        #loadingOverlay {
            position: fixed;
            inset: 0;
            background: rgba(10, 14, 39, 0.85);
            backdrop-filter: blur(8px);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 10001;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.3s ease;
        }

        #loadingOverlay.active {
            opacity: 1;
            pointer-events: all;
        }

        .loader-spinner {
            width: 48px;
            height: 48px;
            border: 4px solid rgba(59, 130, 246, 0.2);
            border-top-color: var(--primary);
            border-radius: 50%;
            animation: spin 0.8s linear infinite;
        }

        #loadingText {
            margin-top: 16px;
            color: var(--text-secondary);
            font-size: 0.9rem;
        }

        /* ========================
           MODAL
           ======================== */
        #modalOverlay {
            position: fixed;
            inset: 0;
            background: rgba(0, 0, 0, 0.7);
            backdrop-filter: blur(4px);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 9999;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.3s ease;
            padding: 20px;
        }

        #modalOverlay.active {
            opacity: 1;
            pointer-events: all;
        }

        .modal {
            background: var(--bg-card);
            border-radius: var(--radius-lg);
            border: 1px solid var(--border);
            width: 100%;
            max-width: 420px;
            max-height: 80vh;
            overflow-y: auto;
            box-shadow: var(--shadow-lg);
            animation: bounceIn 0.4s ease;
        }

        .modal-header {
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 18px 20px;
            border-bottom: 1px solid var(--border);
        }

        .modal-header h3 {
            font-size: 1.1rem;
            font-weight: 600;
        }

        .modal-close {
            background: none;
            border: none;
            color: var(--text-secondary);
            font-size: 1.5rem;
            cursor: pointer;
            width: 32px;
            height: 32px;
            display: flex;
            align-items: center;
            justify-content: center;
            border-radius: 50%;
            transition: background 0.2s;
        }

        .modal-close:hover { background: var(--bg-hover); }

        .modal-body {
            padding: 20px;
        }

        .modal-actions {
            display: flex;
            gap: 10px;
            padding: 16px 20px;
            border-top: 1px solid var(--border);
            justify-content: flex-end;
        }

        /* ========================
           AUTH SCREEN
           ======================== */
        #authScreen {
            display: flex;
            flex-direction: column;
            min-height: 100vh;
            position: relative;
            overflow: hidden;
        }

        .auth-bg {
            position: fixed;
            inset: 0;
            background: var(--gradient-1);
            z-index: -1;
        }

        .auth-bg::before {
            content: '';
            position: absolute;
            top: -50%;
            left: -50%;
            width: 200%;
            height: 200%;
            background: radial-gradient(circle at 30% 20%, rgba(59, 130, 246, 0.15) 0%, transparent 50%),
                        radial-gradient(circle at 70% 80%, rgba(139, 92, 246, 0.1) 0%, transparent 50%);
            animation: pulse 8s ease-in-out infinite;
        }

        .auth-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            padding: 40px 24px;
            animation: fadeIn 0.5s ease;
        }

        .auth-logo {
            text-align: center;
            margin-bottom: 32px;
        }

        .logo-icon {
            font-size: 3.5rem;
            margin-bottom: 8px;
            animation: bounceIn 0.6s ease;
        }

        .logo-text {
            font-size: 1.8rem;
            font-weight: 800;
            background: var(--gradient-2);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
        }

        .logo-text span {
            color: var(--primary-light);
        }

        .logo-tagline {
            color: var(--text-secondary);
            font-size: 0.85rem;
            margin-top: 4px;
        }

        .auth-tabs {
            display: flex;
            background: var(--bg-card);
            border-radius: var(--radius-md);
            padding: 4px;
            margin-bottom: 24px;
            width: 100%;
            max-width: 400px;
            border: 1px solid var(--border);
        }

        .auth-tab {
            flex: 1;
            padding: 10px;
            background: none;
            border: none;
            color: var(--text-secondary);
            font-size: 0.9rem;
            font-weight: 600;
            cursor: pointer;
            border-radius: var(--radius-sm);
            transition: all 0.3s ease;
        }

        .auth-tab.active {
            background: var(--gradient-2);
            color: white;
            box-shadow: var(--shadow-sm);
        }

        .auth-form {
            display: none;
            width: 100%;
            max-width: 400px;
            animation: slideUp 0.3s ease;
        }

        .auth-form.active { display: block; }

        .input-group {
            position: relative;
            margin-bottom: 14px;
        }

        .input-icon {
            position: absolute;
            left: 14px;
            top: 50%;
            transform: translateY(-50%);
            font-size: 1rem;
            pointer-events: none;
        }

        .input-group input,
        .input-group select,
        .input-group textarea {
            width: 100%;
            padding: 14px 14px 14px 44px;
            background: var(--bg-card);
            border: 1px solid var(--border);
            border-radius: var(--radius-md);
            color: var(--text-primary);
            font-size: 0.95rem;
            font-family: inherit;
            transition: all 0.3s ease;
            outline: none;
        }

        .input-group input:focus,
        .input-group select:focus,
        .input-group textarea:focus {
            border-color: var(--primary);
            box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.15);
        }

        .input-group input::placeholder,
        .input-group textarea::placeholder {
            color: var(--text-muted);
        }

        .input-group textarea {
            resize: vertical;
            min-height: 80px;
        }

        .input-group label {
            display: block;
            font-size: 0.8rem;
            color: var(--text-secondary);
            margin-bottom: 6px;
            font-weight: 500;
        }

        .forgot-link {
            background: none;
            border: none;
            color: var(--primary-light);
            font-size: 0.85rem;
            cursor: pointer;
            margin-bottom: 16px;
            padding: 0;
        }

        .auth-divider {
            display: flex;
            align-items: center;
            gap: 12px;
            width: 100%;
            max-width: 400px;
            margin: 20px 0;
            color: var(--text-muted);
            font-size: 0.8rem;
        }

        .auth-divider::before,
        .auth-divider::after {
            content: '';
            flex: 1;
            height: 1px;
            background: var(--border);
        }

        .auth-footer {
            color: var(--text-muted);
            font-size: 0.75rem;
            text-align: center;
            margin-top: 24px;
            max-width: 400px;
        }

        .auth-footer a {
            color: var(--primary-light);
            cursor: pointer;
            text-decoration: underline;
        }

        /* ========================
           BUTTONS
           ======================== */
        .btn {
            display: inline-flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
            padding: 13px 24px;
            border: none;
            border-radius: var(--radius-md);
            font-size: 0.95rem;
            font-weight: 600;
            font-family: inherit;
            cursor: pointer;
            transition: all 0.3s ease;
            outline: none;
            white-space: nowrap;
        }

        .btn:active { transform: scale(0.97); }
        .btn:disabled { opacity: 0.5; cursor: not-allowed; transform: none; }

        .btn-primary {
            background: var(--gradient-2);
            color: white;
            box-shadow: 0 4px 15px rgba(59, 130, 246, 0.35);
        }

        .btn-primary:hover:not(:disabled) {
            box-shadow: 0 6px 25px rgba(59, 130, 246, 0.5);
            transform: translateY(-2px);
        }

        .btn-secondary {
            background: var(--bg-hover);
            color: var(--text-primary);
            border: 1px solid var(--border);
        }

        .btn-success { background: var(--success); color: white; }
        .btn-danger { background: var(--danger); color: white; }
        .btn-warning { background: var(--warning); color: white; }
        .btn-google { background: white; color: #333; border: 1px solid #ddd; }
        .btn-google:hover { background: #f5f5f5; }

        .btn-full { width: 100%; }
        .btn-sm { padding: 8px 16px; font-size: 0.85rem; }
        .btn-xs { padding: 5px 10px; font-size: 0.75rem; border-radius: var(--radius-sm); }

        .btn-link {
            background: none;
            border: none;
            color: var(--primary-light);
            font-size: 0.85rem;
            cursor: pointer;
            font-family: inherit;
        }

        /* ========================
           APP LAYOUT
           ======================== */
        #appScreen {
            display: none;
            flex-direction: column;
            min-height: 100vh;
            background: var(--gradient-1);
        }

        .app-header {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            height: var(--header-height);
            background: rgba(10, 14, 39, 0.95);
            backdrop-filter: blur(20px);
            border-bottom: 1px solid var(--border);
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0 16px;
            z-index: 100;
        }

        .header-btn {
            background: none;
            border: none;
            color: var(--text-primary);
            cursor: pointer;
            width: 40px;
            height: 40px;
            display: flex;
            align-items: center;
            justify-content: center;
            border-radius: var(--radius-sm);
            transition: background 0.2s;
        }

        .header-btn:hover { background: var(--bg-hover); }

        .header-title {
            font-size: 1.1rem;
            font-weight: 700;
            position: absolute;
            left: 50%;
            transform: translateX(-50%);
        }

        .header-right {
            display: flex;
            align-items: center;
            gap: 12px;
        }

        .header-balance {
            font-size: 0.85rem;
            font-weight: 700;
            color: var(--primary-light);
            background: rgba(59, 130, 246, 0.15);
            padding: 6px 12px;
            border-radius: var(--radius-sm);
            border: 1px solid rgba(59, 130, 246, 0.3);
        }

        .header-avatar {
            width: 36px;
            height: 36px;
            border-radius: 50%;
            object-fit: cover;
            border: 2px solid var(--primary);
            cursor: pointer;
        }

        .app-content {
            flex: 1;
            padding: calc(var(--header-height) + 16px) 16px calc(var(--nav-height) + 16px + var(--safe-bottom));
            max-width: 600px;
            margin: 0 auto;
            width: 100%;
        }

        /* ========================
           SIDEBAR
           ======================== */
        .sidebar-overlay {
            position: fixed;
            inset: 0;
            background: rgba(0, 0, 0, 0.6);
            backdrop-filter: blur(4px);
            z-index: 200;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.3s ease;
        }

        .sidebar-overlay.active { opacity: 1; pointer-events: all; }

        .sidebar {
            position: fixed;
            top: 0;
            left: -300px;
            width: 280px;
            height: 100vh;
            background: var(--bg-secondary);
            border-right: 1px solid var(--border);
            z-index: 201;
            transition: left 0.3s ease;
            display: flex;
            flex-direction: column;
            overflow-y: auto;
        }

        .sidebar.open { left: 0; }

        .sidebar-header {
            display: flex;
            align-items: center;
            gap: 10px;
            padding: 20px;
            border-bottom: 1px solid var(--border);
            font-weight: 700;
            font-size: 1.1rem;
        }

        .logo-icon-sm { font-size: 1.5rem; }

        .sidebar-menu {
            flex: 1;
            padding: 12px 8px;
            display: flex;
            flex-direction: column;
            gap: 2px;
        }

        .sidebar-item {
            display: flex;
            align-items: center;
            gap: 12px;
            padding: 12px 16px;
            color: var(--text-secondary);
            text-decoration: none;
            border-radius: var(--radius-md);
            font-size: 0.9rem;
            cursor: pointer;
            transition: all 0.2s ease;
        }

        .sidebar-item:hover {
            background: var(--bg-hover);
            color: var(--text-primary);
        }

        .sidebar-item.admin-item {
            color: var(--warning);
            margin-top: 8px;
            border-top: 1px solid var(--border);
            padding-top: 16px;
        }

        .sidebar-footer {
            padding: 16px;
            border-top: 1px solid var(--border);
        }

        /* ========================
           BOTTOM NAV
           ======================== */
        .bottom-nav {
            position: fixed;
            bottom: 0;
            left: 0;
            right: 0;
            height: calc(var(--nav-height) + var(--safe-bottom));
            padding-bottom: var(--safe-bottom);
            background: rgba(17, 22, 56, 0.97);
            backdrop-filter: blur(20px);
            border-top: 1px solid var(--border);
            display: flex;
            align-items: center;
            justify-content: space-around;
            z-index: 100;
            max-width: 600px;
            margin: 0 auto;
        }

        .nav-item {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 2px;
            background: none;
            border: none;
            color: var(--text-muted);
            cursor: pointer;
            padding: 8px 12px;
            border-radius: var(--radius-sm);
            transition: all 0.3s ease;
            font-family: inherit;
            flex: 1;
        }

        .nav-item.active { color: var(--primary-light); }

        .nav-item.active .nav-icon {
            transform: translateY(-2px);
            filter: drop-shadow(0 0 8px rgba(59, 130, 246, 0.5));
        }

        .nav-icon { font-size: 1.3rem; transition: transform 0.3s ease; }
        .nav-label { font-size: 0.65rem; font-weight: 600; }

        /* ========================
           PAGES
           ======================== */
        .page {
            display: none;
            animation: fadeIn 0.3s ease;
        }

        .page.active { display: block; }

        /* ========================
           HOME PAGE
           ======================== */
        .announcement-banner {
            display: flex;
            align-items: center;
            gap: 10px;
            padding: 12px 16px;
            background: linear-gradient(135deg, rgba(245, 158, 11, 0.15), rgba(239, 68, 68, 0.15));
            border: 1px solid rgba(245, 158, 11, 0.3);
            border-radius: var(--radius-md);
            margin-bottom: 16px;
            font-size: 0.85rem;
            animation: slideUp 0.3s ease;
        }

        .announcement-icon { font-size: 1.2rem; }

        .ad-banner {
            margin-bottom: 16px;
            border-radius: var(--radius-md);
            overflow: hidden;
            min-height: 0;
        }

        .welcome-card {
            display: flex;
            align-items: center;
            justify-content: space-between;
            background: var(--bg-card);
            border: 1px solid var(--border);
            border-radius: var(--radius-lg);
            padding: 16px;
            margin-bottom: 16px;
            animation: slideUp 0.3s ease;
        }

        .welcome-left {
            display: flex;
            align-items: center;
            gap: 12px;
        }

        .welcome-avatar {
            width: 48px;
            height: 48px;
            border-radius: 50%;
            object-fit: cover;
            border: 2px solid var(--primary);
        }

        .welcome-left h3 { font-size: 1rem; font-weight: 700; }
        .welcome-left p { font-size: 0.75rem; color: var(--text-muted); }

        .rank-badge {
            display: flex;
            align-items: center;
            gap: 6px;
            background: rgba(139, 92, 246, 0.15);
            border: 1px solid rgba(139, 92, 246, 0.3);
            padding: 6px 12px;
            border-radius: var(--radius-sm);
            font-size: 0.8rem;
            font-weight: 600;
            color: var(--accent);
        }

        .balance-card {
            background: var(--gradient-3);
            border-radius: var(--radius-lg);
            padding: 24px 20px;
            margin-bottom: 16px;
            position: relative;
            overflow: hidden;
            animation: slideUp 0.4s ease;
            box-shadow: var(--shadow-glow);
        }

        .balance-card::before {
            content: '';
            position: absolute;
            top: -50%;
            right: -30%;
            width: 200px;
            height: 200px;
            background: radial-gradient(circle, rgba(255,255,255,0.1) 0%, transparent 70%);
            border-radius: 50%;
        }

        .balance-main {
            display: flex;
            flex-direction: column;
            margin-bottom: 20px;
        }

        .balance-label { font-size: 0.8rem; color: rgba(255,255,255,0.7); margin-bottom: 4px; }

        .balance-amount {
            font-size: 2.2rem;
            font-weight: 800;
            letter-spacing: -1px;
        }

        .balance-stats {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 12px;
        }

        .balance-stat {
            display: flex;
            flex-direction: column;
            gap: 2px;
        }

        .stat-label { font-size: 0.7rem; color: rgba(255,255,255,0.6); }
        .stat-value { font-size: 0.9rem; font-weight: 700; }

        .quick-actions {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 10px;
            margin-bottom: 16px;
            animation: slideUp 0.5s ease;
        }

        .quick-action {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 6px;
            background: var(--bg-card);
            border: 1px solid var(--border);
            border-radius: var(--radius-md);
            padding: 14px 8px;
            color: var(--text-primary);
            font-size: 0.7rem;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            font-family: inherit;
        }

        .quick-action:hover {
            background: var(--bg-hover);
            transform: translateY(-2px);
            border-color: var(--primary);
        }

        .quick-icon { font-size: 1.4rem; }

        /* ========================
           SECTION CARDS
           ======================== */
        .section-card {
            background: var(--bg-card);
            border: 1px solid var(--border);
            border-radius: var(--radius-lg);
            padding: 18px;
            margin-bottom: 16px;
            animation: slideUp 0.3s ease;
        }

        .section-header {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom: 14px;
        }

        .section-header h3 { font-size: 1rem; font-weight: 700; }

        .badge {
            background: rgba(59, 130, 246, 0.15);
            color: var(--primary-light);
            padding: 4px 10px;
            border-radius: var(--radius-sm);
            font-size: 0.75rem;
            font-weight: 600;
        }

        .streak-badge {
            background: rgba(245, 158, 11, 0.15);
            color: var(--warning);
            padding: 4px 10px;
            border-radius: var(--radius-sm);
            font-size: 0.75rem;
            font-weight: 600;
        }

        /* ========================
           GAMES
           ======================== */
        .games-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 12px;
            margin-bottom: 16px;
        }

        .game-card {
            background: var(--bg-card);
            border: 1px solid var(--border);
            border-radius: var(--radius-lg);
            padding: 18px;
            text-align: center;
            animation: slideUp 0.3s ease;
            transition: all 0.3s ease;
        }

        .game-card:hover {
            border-color: var(--primary);
            transform: translateY(-2px);
            box-shadow: var(--shadow-glow);
        }

        .game-icon { font-size: 2rem; display: block; margin-bottom: 8px; }
        .game-card h4 { font-size: 0.9rem; margin-bottom: 4px; }
        .game-card p { font-size: 0.75rem; color: var(--text-muted); margin-bottom: 12px; }

        /* ========================
           TASKS
           ======================== */
        .task-filters {
            display: flex;
            gap: 8px;
            overflow-x: auto;
            padding-bottom: 8px;
            margin-bottom: 12px;
            -webkit-overflow-scrolling: touch;
        }

        .filter-btn {
            padding: 6px 14px;
            background: var(--bg-secondary);
            border: 1px solid var(--border);
            border-radius: 20px;
            color: var(--text-secondary);
            font-size: 0.8rem;
            font-weight: 600;
            cursor: pointer;
            white-space: nowrap;
            transition: all 0.2s;
            font-family: inherit;
        }

        .filter-btn.active {
            background: var(--gradient-2);
            color: white;
            border-color: transparent;
        }

        .tasks-list {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .task-item {
            display: flex;
            align-items: center;
            gap: 12px;
            padding: 14px;
            background: var(--bg-secondary);
            border: 1px solid var(--border);
            border-radius: var(--radius-md);
            transition: all 0.2s;
        }

        .task-item.completed {
            opacity: 0.6;
            border-color: rgba(16, 185, 129, 0.3);
        }

        .task-icon { font-size: 1.5rem; }

        .task-info { flex: 1; }
        .task-info h4 { font-size: 0.9rem; margin-bottom: 2px; }
        .task-info p { font-size: 0.75rem; color: var(--text-muted); margin-bottom: 4px; }
        .task-reward { font-size: 0.8rem; font-weight: 700; color: var(--success); }

        /* ========================
           WALLET
           ======================== */
        .wallet-card {
            background: var(--gradient-3);
            border-radius: var(--radius-lg);
            padding: 24px 20px;
            margin-bottom: 16px;
            box-shadow: var(--shadow-glow);
        }

        .wallet-label { font-size: 0.8rem; color: rgba(255,255,255,0.7); }
        .wallet-amount { font-size: 2rem; font-weight: 800; display: block; margin: 4px 0 16px; }

        .wallet-stats {
            display: flex;
            gap: 20px;
            flex-wrap: wrap;
        }

        .wallet-stat {
            display: flex;
            flex-direction: column;
            gap: 2px;
            min-width: 80px;
        }

        .wallet-stat span:first-child { font-size: 0.7rem; color: rgba(255,255,255,0.6); }
        .wallet-stat span:last-child { font-size: 0.9rem; font-weight: 700; }

        .conversion-rate {
            font-size: 0.8rem;
            color: var(--text-muted);
            margin-bottom: 12px;
            text-align: center;
        }

        /* ========================
           CONVERT NOTICE
           ======================== */
        .convert-notice {
            display: flex;
            gap: 12px;
            align-items: flex-start;
            background: linear-gradient(135deg, rgba(245, 158, 11, 0.15), rgba(239, 68, 68, 0.12));
            border: 1px solid rgba(245, 158, 11, 0.35);
            border-radius: var(--radius-md);
            padding: 14px;
            margin-bottom: 16px;
            font-size: 0.8rem;
        }

        .convert-notice > span { font-size: 1.2rem; }
        .convert-notice strong { display: block; font-size: 0.88rem; margin-bottom: 2px; }
        .convert-notice p { color: var(--text-secondary); font-size: 0.78rem; margin-bottom: 6px; }

        /* ========================
           TRANSACTIONS
           ======================== */
        .transaction-item {
            display: flex;
            align-items: center;
            gap: 12px;
            padding: 12px 0;
            border-bottom: 1px solid var(--border);
        }

        .transaction-item:last-child { border-bottom: none; }

        .tx-icon {
            width: 36px;
            height: 36px;
            display: flex;
            align-items: center;
            justify-content: center;
            background: var(--bg-secondary);
            border-radius: var(--radius-sm);
            font-size: 1rem;
        }

        .tx-details { flex: 1; display: flex; flex-direction: column; }
        .tx-desc { font-size: 0.85rem; font-weight: 600; }
        .tx-date { font-size: 0.7rem; color: var(--text-muted); }

        .tx-amount { font-size: 0.9rem; font-weight: 700; }
        .tx-amount.positive { color: var(--success); }
        .tx-amount.negative { color: var(--danger); }

        /* ========================
           WITHDRAW
           ======================== */
        .withdraw-info {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
            margin-bottom: 16px;
        }

        .info-item {
            background: var(--bg-card);
            border: 1px solid var(--border);
            border-radius: var(--radius-md);
            padding: 12px;
            text-align: center;
        }

        .info-label { display: block; font-size: 0.7rem; color: var(--text-muted); margin-bottom: 4px; }
        .info-value { font-size: 0.9rem; font-weight: 700; color: var(--primary-light); }

        .method-selector {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
            margin-bottom: 16px;
        }

        .method-btn {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 4px;
            padding: 14px 8px;
            background: var(--bg-secondary);
            border: 2px solid var(--border);
            border-radius: var(--radius-md);
            color: var(--text-secondary);
            font-size: 0.75rem;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s;
            font-family: inherit;
        }

        .method-btn.active {
            border-color: var(--primary);
            background: rgba(59, 130, 246, 0.1);
            color: var(--primary-light);
        }

        .method-btn span:first-child { font-size: 1.3rem; }

        .withdrawal-item {
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 12px 0;
            border-bottom: 1px solid var(--border);
        }

        .withdrawal-item:last-child { border-bottom: none; }

        .w-info { display: flex; flex-direction: column; gap: 2px; }
        .w-method { font-size: 0.8rem; font-weight: 700; }
        .w-number { font-size: 0.75rem; color: var(--text-muted); }
        .w-date { font-size: 0.7rem; color: var(--text-muted); }

        .w-right { display: flex; flex-direction: column; align-items: flex-end; gap: 4px; }
        .w-amount { font-size: 0.9rem; font-weight: 700; }

        .w-status {
            font-size: 0.7rem;
            font-weight: 700;
            padding: 3px 8px;
            border-radius: 10px;
            text-transform: capitalize;
        }

        .status-pending { background: rgba(245, 158, 11, 0.15); color: var(--warning); }
        .status-approved { background: rgba(16, 185, 129, 0.15); color: var(--success); }
        .status-rejected { background: rgba(239, 68, 68, 0.15); color: var(--danger); }

        /* ========================
           REFERRALS
           ======================== */
        .referral-hero {
            text-align: center;
            padding: 30px 20px;
            background: var(--gradient-3);
            border-radius: var(--radius-lg);
            margin-bottom: 16px;
            box-shadow: var(--shadow-glow);
        }

        .referral-icon { font-size: 3rem; display: block; margin-bottom: 8px; }
        .referral-hero h2 { font-size: 1.3rem; margin-bottom: 6px; }
        .referral-hero p { font-size: 0.85rem; color: rgba(255,255,255,0.7); }

        .referral-code-box {
            display: flex;
            align-items: center;
            justify-content: space-between;
            background: var(--bg-secondary);
            border: 2px dashed var(--primary);
            border-radius: var(--radius-md);
            padding: 16px;
            font-size: 1.3rem;
            font-weight: 800;
            letter-spacing: 3px;
            color: var(--primary-light);
        }

        .copy-btn {
            background: var(--primary);
            color: white;
            border: none;
            padding: 8px 14px;
            border-radius: var(--radius-sm);
            font-size: 0.8rem;
            font-weight: 600;
            cursor: pointer;
            font-family: inherit;
        }

        .referral-link-box {
            display: flex;
            gap: 8px;
        }

        .referral-link-box input {
            flex: 1;
            padding: 12px;
            background: var(--bg-secondary);
            border: 1px solid var(--border);
            border-radius: var(--radius-sm);
            color: var(--text-primary);
            font-size: 0.8rem;
        }

        .referral-stats {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 12px;
        }

        .stat-card {
            background: var(--bg-card);
            border: 1px solid var(--border);
            border-radius: var(--radius-lg);
            padding: 20px;
            text-align: center;
        }

        .stat-icon { font-size: 1.5rem; display: block; margin-bottom: 8px; }
        .stat-number { font-size: 1.3rem; font-weight: 800; display: block; color: var(--primary-light); }
        .stat-label { font-size: 0.75rem; color: var(--text-muted); }

        /* ========================
           NOTIFICATIONS
           ======================== */
        .notification-item {
            display: flex;
            gap: 12px;
            padding: 14px 0;
            border-bottom: 1px solid var(--border);
        }

        .notification-item:last-child { border-bottom: none; }
        .notification-item.unread { background: rgba(59, 130, 246, 0.05); margin: 0 -18px; padding: 14px 18px; border-radius: var(--radius-sm); }

        .notif-icon { font-size: 1.3rem; }
        .notif-content h4 { font-size: 0.85rem; margin-bottom: 2px; }
        .notif-content p { font-size: 0.78rem; color: var(--text-secondary); }
        .notif-date { font-size: 0.7rem; color: var(--text-muted); margin-top: 4px; display: block; }

        /* ========================
           PROFILE
           ======================== */
        .profile-header {
            text-align: center;
            padding: 30px 20px;
            background: var(--gradient-3);
            border-radius: var(--radius-lg);
            margin-bottom: 16px;
        }

        .profile-avatar {
            width: 90px;
            height: 90px;
            border-radius: 50%;
            object-fit: cover;
            border: 4px solid rgba(255,255,255,0.2);
            margin-bottom: 12px;
        }

        .profile-header h2 { font-size: 1.2rem; margin-bottom: 4px; }
        .profile-header p { font-size: 0.8rem; color: rgba(255,255,255,0.7); }

        .profile-rank {
            display: inline-block;
            margin-top: 8px;
            padding: 4px 14px;
            background: rgba(255,255,255,0.15);
            border-radius: 20px;
            font-size: 0.8rem;
            font-weight: 700;
        }

        .profile-info { display: flex; flex-direction: column; gap: 12px; }

        .info-row {
            display: flex;
            justify-content: space-between;
            font-size: 0.85rem;
        }

        .info-row span:first-child { color: var(--text-muted); }
        .info-row span:last-child { font-weight: 600; }

        /* ========================
           SUPPORT & LEGAL
           ======================== */
        .support-options { display: flex; flex-direction: column; gap: 10px; margin-top: 14px; }

        .support-item {
            display: flex;
            align-items: center;
            gap: 14px;
            padding: 14px;
            background: var(--bg-secondary);
            border: 1px solid var(--border);
            border-radius: var(--radius-md);
            text-decoration: none;
            color: var(--text-primary);
            transition: all 0.2s;
        }

        .support-item:hover { border-color: var(--primary); }

        .support-item span:first-child { font-size: 1.5rem; }
        .support-item strong { font-size: 0.9rem; display: block; }
        .support-item p { font-size: 0.75rem; color: var(--text-muted); }

        .faq-list { display: flex; flex-direction: column; gap: 14px; margin-top: 12px; }

        .faq-item strong { font-size: 0.9rem; display: block; margin-bottom: 4px; }
        .faq-item p { font-size: 0.8rem; color: var(--text-secondary); line-height: 1.5; }

        .legal-content h2 { font-size: 1.3rem; margin-bottom: 4px; }
        .legal-content h3 { font-size: 1rem; margin: 20px 0 8px; color: var(--primary-light); }
        .legal-content p { font-size: 0.85rem; color: var(--text-secondary); line-height: 1.6; }

        /* ========================
           ADMIN PANEL
           ======================== */
        .admin-header {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom: 16px;
        }

        .admin-header h2 { font-size: 1.2rem; }

        .admin-stats {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 10px;
            margin-bottom: 16px;
        }

        .admin-stat-card {
            background: var(--bg-card);
            border: 1px solid var(--border);
            border-radius: var(--radius-md);
            padding: 14px;
            text-align: center;
        }

        .admin-stat-icon { font-size: 1.3rem; display: block; margin-bottom: 4px; }
        .admin-stat-value { font-size: 1.1rem; font-weight: 800; display: block; color: var(--primary-light); }
        .admin-stat-label { font-size: 0.7rem; color: var(--text-muted); }

        .admin-tabs {
            display: flex;
            gap: 6px;
            overflow-x: auto;
            margin-bottom: 14px;
            padding-bottom: 6px;
            -webkit-overflow-scrolling: touch;
        }

        .admin-tab {
            padding: 8px 16px;
            background: var(--bg-secondary);
            border: 1px solid var(--border);
            border-radius: 20px;
            color: var(--text-secondary);
            font-size: 0.8rem;
            font-weight: 600;
            cursor: pointer;
            white-space: nowrap;
            transition: all 0.2s;
            font-family: inherit;
        }

        .admin-tab.active {
            background: var(--gradient-2);
            color: white;
            border-color: transparent;
        }

        .admin-panel { display: none; animation: fadeIn 0.3s ease; }
        .admin-panel.active { display: block; }

        .admin-withdrawal-item,
        .admin-user-item,
        .admin-task-item,
        .admin-notice-item {
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 12px;
            background: var(--bg-secondary);
            border: 1px solid var(--border);
            border-radius: var(--radius-md);
            margin-bottom: 8px;
            gap: 10px;
            font-size: 0.8rem;
        }

        .admin-user-item { flex-direction: column; align-items: flex-start; }

        .admin-user-info { display: flex; flex-direction: column; gap: 2px; width: 100%; }
        .user-name { font-weight: 700; }
        .user-email { font-size: 0.75rem; color: var(--text-muted); }
        .user-balance { font-size: 0.8rem; color: var(--success); font-weight: 700; }

        .admin-user-actions { display: flex; gap: 6px; flex-wrap: wrap; margin-top: 8px; }

        .admin-actions { display: flex; gap: 4px; margin-top: 4px; }

        .admin-w-info { display: flex; flex-direction: column; gap: 2px; }
        .admin-w-right { display: flex; flex-direction: column; align-items: flex-end; gap: 4px; }

        .search-box { margin-bottom: 12px; }

        .search-box input {
            width: 100%;
            padding: 10px 14px;
            background: var(--bg-secondary);
            border: 1px solid var(--border);
            border-radius: var(--radius-md);
            color: var(--text-primary);
            font-size: 0.85rem;
            outline: none;
            font-family: inherit;
        }

        .search-box input:focus { border-color: var(--primary); }

        .empty-state {
            text-align: center;
            color: var(--text-muted);
            font-size: 0.85rem;
            padding: 24px 0;
        }

        /* ========================
           RESPONSIVE
           ======================== */
        @media (min-width: 601px) {
            .app-content { padding-left: 24px; padding-right: 24px; }
            .bottom-nav { border-radius: var(--radius-lg) var(--radius-lg) 0 0; }
        }

        @media (max-width: 360px) {
            .balance-amount { font-size: 1.8rem; }
            .quick-actions { grid-template-columns: repeat(2, 1fr); }
            .games-grid { grid-template-columns: 1fr; }
            .method-selector { grid-template-columns: 1fr; }
            .withdraw-info { grid-template-columns: 1fr; }
        }

        /* Scrollbar */
        ::-webkit-scrollbar { width: 4px; height: 4px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: var(--bg-hover); border-radius: 4px; }
    </style>
</head>
<body>
    <!-- Toast Container -->
    <div id="toastContainer"></div>

    <!-- Loading Overlay -->
    <div id="loadingOverlay">
        <div class="loader-spinner"></div>
        <div id="loadingText">Loading...</div>
    </div>

    <!-- Modal -->
    <div id="modalOverlay">
        <div class="modal">
            <div class="modal-header">
                <h3 id="modalTitle">Modal</h3>
                <button class="modal-close" onclick="closeModal()">×</button>
            </div>
            <div class="modal-body" id="modalBody"></div>
            <div class="modal-actions" id="modalActions"></div>
        </div>
    </div>

    <!-- Auth Screen -->
    <div id="authScreen"></div>

    <!-- App Screen -->
    <div id="appScreen"></div>

    <!-- Firebase SDK will load as module above -->
    <noscript>
        <p style="text-align:center; padding:40px; color:#fff;">JavaScript is required to use Task & Earn Pro.</p>
    </noscript>
</body>
</html>
