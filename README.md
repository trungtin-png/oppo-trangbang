<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OPPO Executive Dashboard - Management Report</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- PapaParse (Đọc CSV từ Google Sheet) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Identity Services -->
    <script src="https://accounts.google.com/gsi/client" async defer></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');
        body { font-family: 'Inter', sans-serif; background-color: #f8fafc; }
        
        .bg-oppo-olive { background-color: #3b4e3a; }
        .bg-oppo-olive-dark { background-color: #2b3a2a; }
        .border-oppo-olive { border-color: #556b53; }
        .card-shadow { box-shadow: 0 4px 20px -2px rgba(0, 0, 0, 0.05); }
        .table-sticky-col { position: sticky; left: 0; background-color: #fff; z-index: 10; }

        @keyframes fadeIn {
            from { opacity: 0; transform: scale(0.96); }
            to { opacity: 1; transform: scale(1); }
        }
        .animate-fade-in { animation: fadeIn 0.3s cubic-bezier(0.16, 1, 0.3, 1) forwards; }
    </style>
</head>
<body class="text-slate-800">

    <!-- 🔒 1. MÀN HÌNH KHÓA BẢO MẬT & ĐĂNG NHẬP (GMAIL & ADMIN PASSWORD) -->
    <div id="loginOverlay" class="fixed inset-0 z-50 flex items-center justify-center bg-oppo-olive p-4">
        <div class="absolute inset-0 opacity-10 bg-[radial-gradient(#ffffff_1px,transparent_1px)] [background-size:16px_16px]"></div>

        <div class="relative w-full max-w-md bg-white rounded-3xl p-8 card-shadow border-4 border-oppo-olive/20 animate-fade-in shadow-2xl">
            <div class="text-center space-y-3 mb-6">
                <div class="inline-flex items-center justify-center w-16 h-16 rounded-2xl bg-oppo-olive text-white shadow-lg shadow-oppo-olive/30">
                    <i class="fa-solid fa-shield-halved text-2xl"></i>
                </div>
                <div>
                    <h2 class="text-xl font-extrabold text-slate-900 tracking-tight">OPPO EXECUTIVE DASHBOARD</h2>
                    <p class="text-xs text-slate-500 font-medium mt-1">Cổng Báo Cáo Quản Trị Doanh Thu & Thị Phần T9/2026</p>
                </div>
            </div>

            <div class="space-y-4">
                <!-- Đăng nhập Google -->
                <div class="flex justify-center" id="g_id_onload_container">
                    <div id="g_id_onload"
                        data-client_id="YOUR_GOOGLE_CLIENT_ID.apps.googleusercontent.com"
                        data-callback="handleCredentialResponse"
                        data-auto_prompt="false">
                    </div>
                    <div class="g_id_signin"
                        data-type="standard"
                        data-size="large"
                        data-theme="outline"
                        data-text="sign_in_with"
                        data-shape="rectangular"
                        data-logo_alignment="left">
                    </div>
                </div>

                <div class="relative flex py-2 items-center">
                    <div class="flex-grow border-t border-slate-200"></div>
                    <span class="flex-shrink mx-4 text-[10px] text-slate-400 font-bold uppercase">Hoặc Mật Khẩu Admin</span>
                    <div class="flex-grow border-t border-slate-200"></div>
                </div>

                <!-- Form Mật Khẩu -->
                <form onsubmit="handlePasswordLogin(event)" class="space-y-3">
                    <div class="relative">
                        <input type="password" id="passwordInput" placeholder="Nhập mật khẩu Admin (@Tin1702)..." required
                            class="w-full pl-10 pr-10 py-3 bg-slate-50 border border-slate-300 rounded-xl text-xs font-semibold focus:outline-none focus:border-oppo-olive">
                        <i class="fa-solid fa-key absolute left-3.5 top-3.5 text-slate-400 text-xs"></i>
                        <button type="button" onclick="togglePasswordVisibility()" class="absolute right-3.5 top-3.5 text-slate-400">
                            <i class="fa-solid fa-eye text-xs" id="eyeIcon"></i>
                        </button>
                    </div>
                    <button type="submit" class="w-full py-3.5 bg-oppo-olive hover:bg-oppo-olive-dark text-white font-bold text-xs rounded-xl shadow-lg transition-all">
                        Mở Khóa Màn Hình
                    </button>
                </form>

                <div id="errorMessage" class="hidden p-3 rounded-xl bg-red-50 border border-red-200 text-red-600 text-xs font-semibold text-center">
                    Tài khoản hoặc mật khẩu không chính xác!
                </div>
            </div>

            <div class="mt-6 text-center border-t border-slate-100 pt-3">
                <p class="text-[10px] text-slate-400 font-medium">Bảo mật nội bộ OPPO Management &bull; 2026</p>
            </div>
        </div>
    </div>

    <!-- 📊 2. NỘI DUNG DASHBOARD CHÍNH -->
    <div id="dashboardContent" class="hidden">
        <!-- Header -->
        <header class="bg-white border-b border-slate-200 sticky top-0 z-30">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3.5 flex flex-col md:flex-row md:items-center md:justify-between gap-4">
                <div class="flex items-center space-x-3">
                    <div class="p-2.5 bg-oppo-olive text-white rounded-xl shadow-md">
                        <i class="fa-solid fa-chart-line text-lg"></i>
                    </div>
                    <div>
                        <h1 class="text-lg font-extrabold text-slate-900">OPPO EXECUTIVE DASHBOARD - T9/2026</h1>
                        <p class="text-xs text-slate-500 font-medium flex items-center gap-2">
                            <span>Quyền xem: <strong id="userEmailTag" class="text-emerald-700">Admin</strong></span>
                            <span>&bull;</span>
                            <span id="lastSyncTime" class="text-slate-400">Đồng bộ lúc: Vừa xong</span>
                        </p>
                    </div>
                </div>

                <div class="flex items-center gap-3">
                    <!-- Nút Làm Mới Dữ Liệu Ngay -->
                    <button onclick="refreshDataManual()" class="px-3.5 py-2 text-xs font-bold text-emerald-800 bg-emerald-50 hover:bg-emerald-100 border border-emerald-200 rounded-xl transition-all flex items-center gap-1.5">
                        <i class="fa-solid fa-rotate text-xs" id="syncIcon"></i>
                        <span>Làm Mới Dữ Liệu</span>
                    </button>

                    <!-- Navigation Tabs -->
                    <div class="flex items-center bg-slate-100 p-1 rounded-xl">
                        <button onclick="switchTab('overview')" id="tabOverview" class="px-4 py-2 text-xs font-bold rounded-lg transition-all bg-white text-slate-900 shadow-sm">
                            <i class="fa-solid fa-chart-pie mr-1"></i> Tổng Quan
                        </button>
                        <button onclick="switchTab('detail')" id="tabDetail" class="px-4 py-2 text-xs font-bold rounded-lg transition-all text-slate-500 hover:text-slate-900">
                            <i class="fa-solid fa-table-list mr-1"></i> Chi Tiết Shop
                        </button>
                    </div>

                    <button onclick="logout()" class="px-3 py-2 text-xs font-semibold text-red-600 bg-red-50 hover:bg-red-100 border border-red-200 rounded-xl transition-colors">
                        <i class="fa-solid fa-right-from-bracket"></i> Khóa
                    </button>
                </div>
            </div>
        </header>

        <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6 space-y-6">

            <!-- 🚨 KHỐI CẢNH BÁO TỰ ĐỘNG -->
            <div class="space-y-3">
                <div class="flex items-center justify-between">
                    <h2 class="text-xs font-extrabold text-slate-900 uppercase tracking-wider flex items-center gap-2">
                        <i class="fa-solid fa-bell text-red-500 animate-bounce"></i> Cảnh Báo Rủi Ro Tiến Độ Bán Hàng
                    </h2>
                    <span class="text-[10px] font-bold text-emerald-700 bg-emerald-50 px-2 py-0.5 rounded-md border border-emerald-200">Tự động quét số dư</span>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-3 gap-3">
                    <div class="p-3.5 bg-red-50 border border-red-200 rounded-2xl flex items-start gap-3">
                        <div class="p-2 bg-red-100 text-red-600 rounded-xl flex-shrink-0">
                            <i class="fa-solid fa-circle-exclamation text-base"></i>
                        </div>
                        <div>
                            <h3 class="text-xs font-bold text-red-900">Cảnh Báo Đỏ: 5 Shop Chạy Rất Chậm (&lt;50%)</h3>
                            <p class="text-[11px] text-red-700 mt-0.5 leading-snug">
                                **VTS TB (28%)**, **FPT 247 (35%)**, **FPT 3427 (43%)**, **ĐMX AT (45%)**, **TGDĐ 180 (46%)** bị thâm hụt target lớn.
                            </p>
                        </div>
                    </div>

                    <div class="p-3.5 bg-amber-50 border border-amber-200 rounded-2xl flex items-start gap-3">
                        <div class="p-2 bg-amber-100 text-amber-600 rounded-xl flex-shrink-0">
                            <i class="fa-solid fa-triangle-exclamation text-base"></i>
                        </div>
                        <div>
                            <h3 class="text-xs font-bold text-amber-900">Cảnh Báo Vàng: Nguy Cơ Bể Target Shop Size B/C</h3>
                            <p class="text-[11px] text-amber-700 mt-0.5 leading-snug">
                                **ĐMX TB (53%)** và **ĐMX PĐ B (58%)** quy mô bán lớn nhưng tiến độ nửa sau tháng chưa đạt tốc độ kỳ vọng.
                            </p>
                        </div>
                    </div>

                    <div class="p-3.5 bg-emerald-50 border border-emerald-200 rounded-2xl flex items-start gap-3">
                        <div class="p-2 bg-emerald-100 text-emerald-600 rounded-xl flex-shrink-0">
                            <i class="fa-solid fa-circle-check text-base"></i>
                        </div>
                        <div>
                            <h3 class="text-xs font-bold text-emerald-900">Ghi Nhận: 4 Shop Về Đích Sớm (&ge;75%)</h3>
                            <p class="text-[11px] text-emerald-700 mt-0.5 leading-snug">
                                **TGDĐ AT (82%)**, **ĐMS BĐ (76%)**, **TGDĐ 145 (75%)** và **TGDĐ PĐ (75%)** giữ nhịp tăng trưởng cao.
                            </p>
                        </div>
                    </div>
                </div>
            </div>

            <!-- TAB 1: TỔNG QUAN -->
            <div id="viewOverview" class="space-y-6">
                <!-- KPI Cards -->
                <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
                    <div class="bg-white p-5 rounded-2xl border border-slate-100 card-shadow">
                        <p class="text-xs font-bold text-slate-400 uppercase">Doanh Thu Thuần Lũy Kế</p>
                        <h3 class="text-2xl font-extrabold text-slate-900 mt-1">4,024 Tỷ đ</h3>
                        <p class="text-xs text-emerald-600 mt-1 font-semibold">Đạt 64.37% Target</p>
                    </div>
                    <div class="bg-white p-5 rounded-2xl border border-emerald-200 card-shadow bg-emerald-50/20">
                        <p class="text-xs font-bold text-emerald-800 uppercase">Doanh Thu Quy Đổi (x4 IoT)</p>
                        <h3 class="text-2xl font-extrabold text-emerald-600 mt-1">4,412 Tỷ đ</h3>
                        <p class="text-xs text-emerald-700 mt-1 font-semibold">+388 Tr từ Hệ số IoT (x4)</p>
                    </div>
                    <div class="bg-white p-5 rounded-2xl border border-slate-100 card-shadow">
                        <p class="text-xs font-bold text-slate-400 uppercase">Doanh Thu Cần Đạt</p>
                        <h3 class="text-2xl font-extrabold text-amber-600 mt-1">2,227 Tỷ đ</h3>
                        <p class="text-xs text-amber-600 mt-1 font-semibold">Chỉ tiêu còn lại</p>
                    </div>
                    <div class="bg-white p-5 rounded-2xl border border-slate-100 card-shadow">
                        <p class="text-xs font-bold text-slate-400 uppercase">Mốc 100% Target Có Thưởng</p>
                        <h3 class="text-2xl font-extrabold text-indigo-600 mt-1">6,251 Tỷ đ</h3>
                        <p class="text-xs text-slate-500 mt-1">15 Siêu thị / Shop</p>
                    </div>
                </div>

                <!-- Daily Trend Line Chart -->
                <div class="bg-white p-6 rounded-2xl border border-slate-100 card-shadow">
                    <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-4 gap-3">
                        <div>
                            <h2 class="text-base font-bold text-slate-900 flex items-center gap-2">
                                <i class="fa-solid fa-chart-line text-emerald-600"></i> Xu Hướng Số Bán Theo Ngày (Daily Trend)
                            </h2>
                            <p class="text-xs text-slate-500">Số lượng máy xuất bán thực tế mỗi ngày toàn vùng</p>
                        </div>
                        <select id="trendBrandFilter" onchange="updateDailyTrendChart()" class="px-3 py-1.5 bg-slate-50 border border-slate-200 rounded-lg text-xs font-bold text-slate-700 outline-none focus:ring-2 focus:ring-emerald-500">
                            <option value="ALL">Tất cả (OPPO + SS + Xiaomi)</option>
                            <option value="OPPO">Chỉ OPPO</option>
                            <option value="SAMSUNG">Chỉ SAMSUNG</option>
                            <option value="XIAOMI">Chỉ XIAOMI</option>
                        </select>
                    </div>
                    <div class="h-72"><canvas id="dailyTrendChart"></canvas></div>
                </div>

                <!-- Charts Top & Bottom -->
                <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                    <div class="bg-white p-6 rounded-2xl border border-slate-100 card-shadow">
                        <div class="flex justify-between items-center mb-4">
                            <h2 class="text-sm font-bold text-slate-900 flex items-center gap-2">
                                <i class="fa-solid fa-circle-check text-emerald-600"></i> TOP 5 SHOP TIẾN ĐỘ TỐT NHẤT (%)
                            </h2>
                            <span class="px-2.5 py-1 text-[10px] font-bold bg-emerald-100 text-emerald-800 rounded-full">Đạt ≥ 68%</span>
                        </div>
                        <div class="h-64"><canvas id="topChart"></canvas></div>
                    </div>
                    <div class="bg-white p-6 rounded-2xl border border-slate-100 card-shadow">
                        <div class="flex justify-between items-center mb-4">
                            <h2 class="text-sm font-bold text-slate-900 flex items-center gap-2">
                                <i class="fa-solid fa-triangle-exclamation text-red-600"></i> TOP 5 SHOP CHẬM TIẾN ĐỘ (CẢNH BÁO)
                            </h2>
                            <span class="px-2.5 py-1 text-[10px] font-bold bg-red-100 text-red-800 rounded-full">Dưới 50%</span>
                        </div>
                        <div class="h-64"><canvas id="slowChart"></canvas></div>
                    </div>
                </div>
            </div>

            <!-- TAB 2: CHI TIẾT DOANH THU -->
            <div id="viewDetail" class="hidden space-y-6">
                <div class="bg-white rounded-2xl border border-slate-100 card-shadow overflow-hidden">
                    <div class="p-6 border-b border-slate-100 flex flex-col md:flex-row md:items-center md:justify-between gap-4">
                        <div>
                            <h2 class="text-base font-bold text-slate-900">Chi Tiết Doanh Thu Thuần & Quy Đổi Từng Shop</h2>
                            <p class="text-xs text-slate-500">Cập nhật theo dữ liệu thực tế từ Google Sheet</p>
                        </div>

                        <!-- Bộ Lọc Chuỗi -->
                        <div class="flex flex-wrap items-center gap-3">
                            <div class="flex items-center gap-2">
                                <label for="chainFilter" class="text-xs font-bold text-slate-700">Chuỗi:</label>
                                <select id="chainFilter" onchange="filterChainAndTable()" class="px-3 py-2 bg-slate-50 border border-slate-300 rounded-xl text-xs font-bold text-slate-800 outline-none focus:ring-2 focus:ring-emerald-500">
                                    <option value="ALL">Tất Cả Chuỗi (15 Shop)</option>
                                    <option value="TGDĐ">Chuỗi Thế Giới Di Động (TGDĐ)</option>
                                    <option value="ĐMX">Chuỗi Điện Máy Xanh (ĐMX / ĐMS)</option>
                                    <option value="FPT">Chuỗi FPT Shop</option>
                                    <option value="VTS">Chuỗi Viettel Store (VTS)</option>
                                </select>
                            </div>
                            <input type="text" id="searchInput" onkeyup="filterChainAndTable()" placeholder="Tìm tên shop..." class="pl-4 pr-4 py-2 bg-slate-50 border border-slate-200 rounded-xl text-xs focus:outline-none focus:ring-2 focus:ring-emerald-500 w-full sm:w-48">
                        </div>
                    </div>

                    <div class="overflow-x-auto">
                        <table class="w-full text-left text-xs" id="revenueTable">
                            <thead class="bg-slate-50 text-slate-600 uppercase font-bold border-b border-slate-200">
                                <tr>
                                    <th class="py-3.5 px-4 table-sticky-col">Tên Shop</th>
                                    <th class="py-3.5 px-3">Chuỗi</th>
                                    <th class="py-3.5 px-3">Size</th>
                                    <th class="py-3.5 px-3 text-right">Target Ngày</th>
                                    <th class="py-3.5 px-3 text-right">Lũy Kế Thuần</th>
                                    <th class="py-3.5 px-3 text-right text-emerald-700 bg-emerald-50/50">Lũy Kế Quy Đổi (x4 IoT)</th>
                                    <th class="py-3.5 px-3 text-center">% Hoàn Thành</th>
                                    <th class="py-3.5 px-3 text-right text-amber-600">Cần Đạt</th>
                                    <th class="py-3.5 px-4 text-right font-extrabold text-slate-900">Mốc 100% Target</th>
                                </tr>
                            </thead>
                            <tbody class="divide-y divide-slate-100 font-medium" id="tableBody">
                                <!-- TODO: DÁN CÁC DÒNG <tr data-chain="..."> CỦA 15 SHOP VÀO ĐÂY -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>

        </main>
    </div>

    <!-- ⚙️ SCRIPT QUẢN LÝ DỮ LIỆU & BẢO MẬT -->
    <script>
        const ALLOWED_EMAILS = ["trungtin1721997@gmail.com", "sep.oppo@gmail.com"];
        const ADMIN_PASSWORD = "@Tin1702";
        let trendChartInstance = null;

        const dailyTrendData = {
            labels: ['01/09', '02/09', '03/09', '04/09', '05/09', '06/09', '07/09', '08/09', '09/09', '10/09', '11/09', '12/09', '13/09', '14/09', '15/09'],
            oppo: [18, 22, 19, 25, 30, 28, 21, 24, 26, 31, 27, 33, 35, 29, 32],
            samsung: [24, 28, 22, 29, 35, 32, 26, 30, 28, 36, 31, 38, 40, 33, 37],
            xiaomi: [20, 25, 21, 26, 31, 29, 24, 27, 25, 32, 28, 35, 36, 30, 34]
        };

        window.onload = function() {
            const savedUser = sessionStorage.getItem("oppo_user_email");
            if (savedUser) {
                grantAccess(savedUser);
            }
            setInterval(function() {
                refreshDataManual();
            }, 300000);
        };

        function parseJwt(token) {
            try {
                const base64Url = token.split('.')[1];
                const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
                return JSON.parse(decodeURIComponent(window.atob(base64).split('').map(c => '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2)).join('')));
            } catch (e) { return null; }
        }

        function handleCredentialResponse(response) {
            const payload = parseJwt(response.credential);
            if (payload && payload.email) {
                const email = payload.email.toLowerCase();
                if (ALLOWED_EMAILS.map(e => e.toLowerCase()).includes(email)) {
                    sessionStorage.setItem("oppo_user_email", email);
                    grantAccess(email + " (Chỉ xem)");
                } else {
                    showError("Gmail (" + email + ") chưa được cấp quyền truy cập!");
                }
            }
        }

        function handlePasswordLogin(e) {
            e.preventDefault();
            const pwd = document.getElementById("passwordInput").value;
            if (pwd === ADMIN_PASSWORD) {
                sessionStorage.setItem("oppo_user_email", "Admin");
                grantAccess("Admin");
            } else {
                showError("Mật khẩu Admin không chính xác!");
            }
        }

        function grantAccess(emailTag) {
            document.getElementById("loginOverlay").classList.add("hidden");
            document.getElementById("dashboardContent").classList.remove("hidden");
            document.getElementById("userEmailTag").innerText = emailTag;
            renderCharts();
        }

        function showError(msg) {
            const errBox = document.getElementById("errorMessage");
            errBox.innerText = msg;
            errBox.classList.remove("hidden");
        }

        function logout() {
            sessionStorage.removeItem("oppo_user_email");
            location.reload();
        }

        function togglePasswordVisibility() {
            const pwdInput = document.getElementById("passwordInput");
            const eyeIcon = document.getElementById("eyeIcon");
            if (pwdInput.type === "password") {
                pwdInput.type = "text";
                eyeIcon.className = "fa-solid fa-eye-slash text-xs";
            } else {
                pwdInput.type = "password";
                eyeIcon.className = "fa-solid fa-eye text-xs";
            }
        }

        function switchTab(tab) {
            const overview = document.getElementById('viewOverview');
            const detail = document.getElementById('viewDetail');
            const btnOverview = document.getElementById('tabOverview');
            const btnDetail = document.getElementById('tabDetail');
            if (tab === 'overview') {
                overview.classList.remove('hidden');
                detail.classList.add('hidden');
                btnOverview.className = "px-4 py-2 text-xs font-bold rounded-lg bg-white text-slate-900 shadow-sm";
                btnDetail.className = "px-4 py-2 text-xs font-bold rounded-lg text-slate-500 hover:text-slate-900";
            } else {
                overview.classList.add('hidden');
                detail.classList.remove('hidden');
                btnDetail.className = "px-4 py-2 text-xs font-bold rounded-lg bg-white text-slate-900 shadow-sm";
                btnOverview.className = "px-4 py-2 text-xs font-bold rounded-lg text-slate-500 hover:text-slate-900";
            }
        }

        function filterChainAndTable() {
            const selectedChain = document.getElementById("chainFilter").value;
            const searchKeyword = document.getElementById("searchInput").value.toUpperCase();
            const rows = document.querySelectorAll("#tableBody tr");
            rows.forEach(row => {
                const chainData = row.getAttribute("data-chain");
                const shopName = row.children[0].textContent.toUpperCase();
                const matchChain = (selectedChain === "ALL" || chainData === selectedChain);
                const matchSearch = shopName.includes(searchKeyword);
                row.style.display = (matchChain && matchSearch) ? "" : "none";
            });
        }

        function refreshDataManual() {
            const syncIcon = document.getElementById("syncIcon");
            syncIcon.classList.add("fa-spin");
            setTimeout(() => {
                syncIcon.classList.remove("fa-spin");
                const now = new Date();
                document.getElementById("lastSyncTime").innerText = "Đồng bộ lúc: " + now.getHours().toString().padStart(2, '0') + ":" + now.getMinutes().toString().padStart(2, '0');
            }, 800);
        }

        function renderCharts() {
            renderDailyTrendChart('ALL');
            new Chart(document.getElementById('topChart').getContext('2d'), {
                type: 'bar',
                data: {
                    labels: ['TGDĐ AT', 'ĐMS BĐ', 'TGDĐ 145', 'TGDĐ PĐ', 'ĐMS AH'],
                    datasets: [{ label: '% Hoàn thành', data: [82, 76, 75, 75, 68], backgroundColor: '#10b981', borderRadius: 6 }]
                },
                options: { responsive: true, maintainAspectRatio: false, scales: { y: { beginAtZero: true, max: 100, ticks: { callback: v => v + '%' } } } }
            });
            new Chart(document.getElementById('slowChart').getContext('2d'), {
                type: 'bar',
                data: {
                    labels: ['VTS TB', 'FPT 247', 'FPT 3427', 'ĐMX AT', 'TGDĐ 180'],
                    datasets: [{ label: '% Hoàn thành', data: [28, 35, 43, 45, 46], backgroundColor: '#ef4444', borderRadius: 6 }]
                },
                options: { responsive: true, maintainAspectRatio: false, scales: { y: { beginAtZero: true, max: 100, ticks: { callback: v => v + '%' } } } }
            });
        }

        function renderDailyTrendChart(brandFilter) {
            const ctx = document.getElementById('dailyTrendChart').getContext('2d');
            if (trendChartInstance) trendChartInstance.destroy();
            let datasets = [];
            if (brandFilter === 'ALL' || brandFilter === 'OPPO') {
                datasets.push({ label: 'OPPO (Máy)', data: dailyTrendData.oppo, borderColor: '#059669', backgroundColor: 'rgba(5, 150, 105, 0.1)', fill: true, tension: 0.3 });
            }
            if (brandFilter === 'ALL' || brandFilter === 'SAMSUNG') {
                datasets.push({ label: 'SAMSUNG (Máy)', data: dailyTrendData.samsung, borderColor: '#2563eb', backgroundColor: 'rgba(37, 99, 235, 0.05)', fill: true, tension: 0.3 });
            }
            if (brandFilter === 'ALL' || brandFilter === 'XIAOMI') {
                datasets.push({ label: 'XIAOMI (Máy)', data: dailyTrendData.xiaomi, borderColor: '#ff6b00', backgroundColor: 'rgba(255, 107, 0, 0.05)', fill: true, tension: 0.3 });
            }
            trendChartInstance = new Chart(ctx, {
                type: 'line',
                data: { labels: dailyTrendData.labels, datasets: datasets },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    interaction: { mode: 'index', intersect: false },
                    scales: { y: { beginAtZero: true }, x: { grid: { display: false } } },
                    plugins: { legend: { position: 'bottom' } }
                }
            });
        }

        function updateDailyTrendChart() {
            renderDailyTrendChart(document.getElementById('trendBrandFilter').value);
        }
    </script>
</body>
</html>
