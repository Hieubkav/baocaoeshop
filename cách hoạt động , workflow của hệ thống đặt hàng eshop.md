# 🛒 MShopKeeper eShop - Workflow Khách Hàng & Hệ Thống Đặt Hàng

## 📋 Tổng quan hệ thống

Hệ thống eShop MShopKeeper được tích hợp hoàn toàn với API MShopKeeper, cung cấp trải nghiệm mua sắm trực tuyến với dữ liệu thời gian thực từ hệ thống POS.

### 🎯 Các thành phần chính:

- **Khách hàng (MShopKeeperCustomer)**: Quản lý thông tin khách hàng từ API
- **Hàng hóa (MShopKeeperInventoryItem)**: Sản phẩm với tồn kho thời gian thực
- **Danh mục (MShopKeeperCategory)**: Phân loại sản phẩm theo cây thư mục
- **Giỏ hàng (MShopKeeperCart)**: Quản lý đơn hàng trước khi đặt
- **Điểm thành viên (MShopKeeperCustomerPoint)**: Hệ thống tích điểm

---

## 🔄 1. WORKFLOW ĐỒNG BỘ DỮ LIỆU

### 1.1 Sync Commands

```bash
# Đồng bộ khách hàng
php artisan mshopkeeper:sync-customers

# Đồng bộ danh mục sản phẩm  
php artisan mshopkeeper:sync-categories

# Đồng bộ hàng hóa và tồn kho
php artisan mshopkeeper:sync-inventory-items --include-inventory

# Đồng bộ điểm thành viên
php artisan mshopkeeper:sync-customer-points
```

### 1.2 Quy trình Sync tự động

```
API MShopKeeper → Laravel Commands → Database → Cache → Frontend
     ↓              ↓                  ↓         ↓        ↓
  Real-time     Scheduled Sync    Local Storage  Fast    User
    Data         (Cron Jobs)      (MySQL)      Access   Experience
```

### 1.3 Bảo vệ dữ liệu tùy chỉnh

- `is_visible`: Ẩn/hiện sản phẩm (không bị reset khi sync)
- `is_featured`: Đánh dấu sản phẩm nổi bật (không bị reset khi sync)
- Dữ liệu tùy chỉnh được bảo vệ khỏi bị ghi đè

---

## 👥 2. WORKFLOW KHÁCH HÀNG

### 2.1 Thông tin khách hàng

**Model**: `MShopKeeperCustomer`
**Bảng**: `mshopkeeper_customers`

```php
// Cấu trúc dữ liệu khách hàng
[
    'mshopkeeper_id' => 'CUST001',
    'code' => 'KH001',
    'name' => 'Nguyễn Văn A',
    'tel' => '0901234567',
    'normalized_tel' => '901234567',
    'standard_tel' => '+84901234567',
    'email' => 'customer@example.com',
    'gender' => 1, // 1: Nam, 2: Nữ
    'addr' => 'Địa chỉ đầy đủ',
    'membership_code' => 'VIP001',
    'member_level_name' => 'Thành viên VIP'
]
```

### 2.2 Quy trình quản lý khách hàng

```
1. Sync từ API → 2. Lưu Database → 3. Hiển thị Admin → 4. Tracking điểm
      ↓                ↓                ↓                ↓
   Real-time        Local Cache      Management      Loyalty Program
```

### 2.3 Tính năng khách hàng

- ✅ **Sync tự động**: Dữ liệu luôn cập nhật từ API
- ✅ **Chuẩn hóa SĐT**: Tự động format số điện thoại
- ✅ **Phân cấp thành viên**: VIP, thường, mới
- ✅ **Tracking điểm**: Tích hợp hệ thống điểm thưởng
- ✅ **Chỉ đọc**: Không chỉnh sửa từ admin (đảm bảo tính nhất quán)

---

## 📦 3. WORKFLOW HÀNG HÓA & TỒN KHO

### 3.1 Cấu trúc sản phẩm

**Model**: `MShopKeeperInventoryItem`
**Bảng**: `mshopkeeper_inventory_items`

```php
// Thông tin sản phẩm
[
    'mshopkeeper_id' => 'ITEM001',
    'code' => 'SP001',
    'name' => 'Sản phẩm A',
    'description' => 'Mô tả chi tiết',
    'unit_price' => 100000,
    'category_id' => 'CAT001',
    'inactive' => false,
    'is_item' => true,
    'is_visible' => true,    // Tùy chỉnh hiển thị
    'is_featured' => false,  // Sản phẩm nổi bật
    'image_url' => 'path/to/image.jpg'
]
```

### 3.2 Tồn kho thời gian thực

**Model**: `MShopKeeperInventoryStock`
**Bảng**: `mshopkeeper_inventory_stocks`

```php
// Thông tin tồn kho
[
    'inventory_item_id' => 'ITEM001',
    'location_id' => 'LOC001',
    'on_hand' => 50,        // Số lượng tồn
    'available' => 45,      // Có thể bán
    'committed' => 5,       // Đã cam kết
    'on_order' => 10        // Đang đặt hàng
]
```

### 3.3 Quy trình quản lý hàng hóa

```
API Sync → Database → Admin Panel → Frontend Display
    ↓         ↓           ↓             ↓
Real-time  Local      Management    Customer
  Data     Storage    (Visibility)   Experience
```

### 3.4 Tính năng hàng hóa

- ✅ **Sync tự động**: Giá và tồn kho cập nhật liên tục
- ✅ **Phân loại**: Hàng hóa, Combo, Dịch vụ
- ✅ **Ẩn/hiện**: Tùy chỉnh hiển thị không bị reset
- ✅ **Sản phẩm nổi bật**: Đánh dấu để hiển thị trang chủ
- ✅ **Multi-location**: Quản lý tồn kho nhiều kho
- ✅ **Parent-Child**: Hỗ trợ sản phẩm có biến thể

---

## 🗂️ 4. WORKFLOW DANH MỤC SẢN PHẨM

### 4.1 Cấu trúc danh mục

**Model**: `MShopKeeperCategory`
**Bảng**: `mshopkeeper_categories`

```php
// Cấu trúc cây danh mục
[
    'mshopkeeper_id' => 'CAT001',
    'code' => 'DM001',
    'name' => 'Điện tử',
    'description' => 'Sản phẩm điện tử',
    'grade' => 1,                    // Cấp độ trong cây
    'parent_mshopkeeper_id' => null, // Danh mục cha
    'is_leaf' => false,              // Node lá hay nhánh
    'sort_order' => 1,               // Thứ tự sắp xếp
    'inactive' => false
]
```

### 4.2 Relationships trong cây danh mục

```php
// Parent-Child relationships
Category::with(['parent', 'children', 'descendants'])
    ->where('inactive', false)
    ->orderBy('sort_order')
    ->get();
```

### 4.3 Quy trình quản lý danh mục

```
API Hierarchy → Build Tree → Database → Admin Tree View → Frontend Menu
      ↓            ↓           ↓           ↓                ↓
   Real-time    Relationships Local     Management      Navigation
   Structure    Processing   Storage    Interface       Experience
```

### 4.4 Tính năng danh mục

- ✅ **Cây phân cấp**: Unlimited levels, parent-child relationships
- ✅ **Sync hierarchy**: Tự động xây dựng cấu trúc cây
- ✅ **Tree view**: Hiển thị cây trong admin với stats
- ✅ **Breadcrumb**: Đường dẫn đầy đủ từ root
- ✅ **Leaf detection**: Tự động phát hiện node lá
- ✅ **Sort order**: Sắp xếp theo thứ tự từ API

---

## 🛒 5. WORKFLOW GIỎ HÀNG & ĐẶT HÀNG

### 5.1 Cấu trúc giỏ hàng

**Model**: `MShopKeeperCart`
**Bảng**: `mshopkeeper_carts`

```php
// Giỏ hàng
[
    'customer_id' => 1,
    'inventory_item_id' => 'ITEM001',
    'quantity' => 2,
    'unit_price' => 100000,
    'total_price' => 200000,
    'notes' => 'Ghi chú đặc biệt'
]
```

### 5.2 Customer Journey - Quy trình mua hàng

```
1. Duyệt sản phẩm → 2. Thêm giỏ hàng → 3. Xem giỏ hàng → 4. Liên hệ đặt hàng
        ↓                   ↓                ↓                ↓
   Product Catalog      Add to Cart      Review Cart      Contact Form
   /kho-hang           AJAX Request      /gio-hang       /gio-hang/lien-he
```

### 5.3 Routes chính của hệ thống

```php
// Frontend Routes
'/kho-hang'                    // Trang chủ kho hàng
'/kho-hang/loai/hang-hoa'      // Hàng hoá
'/kho-hang/loai/combo'         // Combo sản phẩm  
'/kho-hang/loai/dich-vu'       // Dịch vụ
'/kho-hang/noi-bat'            // Sản phẩm nổi bật
'/kho-hang/tim-kiem'           // Tìm kiếm
'/kho-hang/san-pham/{code}'    // Chi tiết sản phẩm
'/gio-hang'                    // Giỏ hàng
'/gio-hang/lien-he'            // Liên hệ đặt hàng

// Admin Routes  
'/admin/mshopkeeper-inventory-items'     // Quản lý hàng hóa
'/admin/m-shop-keeper-categories'        // Quản lý danh mục
'/admin/mshopkeeper-customers'           // Quản lý khách hàng
'/admin/m-shop-keeper-carts'             // Xem giỏ hàng
'/admin/mshopkeeper-customer-points'     // Điểm thành viên
```

### 5.4 Quy trình đặt hàng chi tiết

#### Bước 1: Duyệt sản phẩm

```
Trang chủ kho hàng → Chọn danh mục → Xem danh sách → Chi tiết sản phẩm
      ↓                  ↓              ↓              ↓
  Category Menu      Filter/Search   Product Grid   Product Detail
  (Tree structure)   (Real-time)    (Pagination)   (Full info)
```

#### Bước 2: Thêm vào giỏ hàng

```javascript
// AJAX Request thêm sản phẩm
POST /api/cart/add
{
    "inventory_item_id": "ITEM001",
    "quantity": 2,
    "notes": "Ghi chú"
}

// Response
{
    "success": true,
    "message": "Đã thêm vào giỏ hàng",
    "cart_count": 3,
    "cart_total": 500000
}
```

#### Bước 3: Quản lý giỏ hàng

```
Xem giỏ hàng → Cập nhật số lượng → Xóa sản phẩm → Tính tổng tiền
     ↓              ↓                ↓              ↓
  /gio-hang    AJAX Update      AJAX Delete    Auto Calculate
```

#### Bước 4: Liên hệ đặt hàng

```
Giỏ hàng → Click "Liên hệ" → Form thông tin → Gửi yêu cầu
    ↓           ↓                ↓              ↓
Cart Review  /gio-hang/lien-he  Customer Info  Contact Admin
```

### 5.5 Tính năng giỏ hàng

- ✅ **Session-based**: Không cần đăng nhập
- ✅ **AJAX operations**: Thêm/sửa/xóa không reload trang
- ✅ **Real-time total**: Tính tổng tiền tự động
- ✅ **Stock validation**: Kiểm tra tồn kho khi thêm
- ✅ **Persistent**: Giữ giỏ hàng qua sessions
- ✅ **Admin tracking**: Admin có thể xem tất cả giỏ hàng

---

## 💎 6. HỆ THỐNG ĐIỂM THÀNH VIÊN

### 6.1 Cấu trúc điểm thưởng

**Model**: `MShopKeeperCustomerPoint`
**Bảng**: `mshopkeeper_customer_points`

```php
// Điểm thành viên
[
    'customer_mshopkeeper_id' => 'CUST001',
    'customer_name' => 'Nguyễn Văn A',
    'customer_code' => 'KH001',
    'total_points' => 1500,
    'available_points' => 1200,
    'used_points' => 300,
    'expired_points' => 0
]
```

### 6.2 Quy trình tích điểm

```
Mua hàng → Tích điểm → Sync API → Update Database → Hiển thị Admin
    ↓         ↓          ↓           ↓              ↓
Purchase   Earn Points  Real-time   Local Storage  Management
Activity   (POS)        Sync        (Cache)        Interface
```

### 6.3 Tính năng điểm thưởng

- ✅ **Sync tự động**: Điểm được cập nhật từ POS
- ✅ **Multi-status**: Available, Used, Expired
- ✅ **Customer linking**: Liên kết với thông tin khách hàng
- ✅ **Admin dashboard**: Thống kê tổng điểm hệ thống
- ✅ **History tracking**: Lịch sử tích/tiêu điểm

---

## 📊 7. ADMIN DASHBOARD & QUẢN LÝ

### 7.1 Filament Resources

```php
// Các Resource chính
MShopKeeperCustomerResource::class         // Khách hàng
MShopKeeperInventoryItemResource::class    // Hàng hóa  
MShopKeeperCategoryResource::class         // Danh mục
MShopKeeperCartResource::class             // Giỏ hàng
MShopKeeperCustomerPointResource::class    // Điểm thành viên
```

### 7.2 Tính năng Admin Panel

- ✅ **Sync actions**: Nút sync dữ liệu từ API
- ✅ **Statistics**: Thống kê tổng quan trong subheading
- ✅ **View-only**: Dữ liệu chỉ đọc, không chỉnh sửa
- ✅ **Filters**: Lọc theo trạng thái, loại, thời gian
- ✅ **Search**: Tìm kiếm toàn văn
- ✅ **Export**: Xuất dữ liệu Excel/CSV
- ✅ **Bulk actions**: Thao tác hàng loạt
- ✅ **Modals**: View chi tiết, sync stats, guides

### 7.3 Header Actions mẫu

```php
// Sync Action
Actions\Action::make('sync')
    ->label('Sync từ API')
    ->icon('heroicon-o-arrow-path')
    ->action(function () {
        Artisan::call('mshopkeeper:sync-customers');
        // Show notification with stats
    });

// Stats Modal
Actions\Action::make('sync_stats')
    ->label('Thống kê sync')
    ->modalContent(view('admin.sync-stats'));
```

---

## 🔧 8. TECHNICAL IMPLEMENTATION

### 8.1 Database Schema Overview

```sql
-- Khách hàng
mshopkeeper_customers (15 fields)
├── Basic info: name, tel, email, gender
├── Address: addr, province, district, commune  
├── Membership: code, level_id, level_name
└── Sync: last_synced_at, sync_status, raw_data

-- Hàng hóa
mshopkeeper_inventory_items (20+ fields)
├── Product info: code, name, description, unit_price
├── Category: category_id, category_name
├── Status: inactive, is_item, is_visible, is_featured
├── Hierarchy: parent_id (for variants)
└── Sync: last_synced_at, sync_status, raw_data

-- Tồn kho  
mshopkeeper_inventory_stocks (10 fields)
├── Stock info: on_hand, available, committed
├── Location: location_id, location_name
└── Sync: last_synced_at, sync_status

-- Danh mục
mshopkeeper_categories (15 fields)
├── Category info: code, name, description, grade
├── Hierarchy: parent_id, is_leaf, sort_order
└── Sync: last_synced_at, sync_status, raw_data

-- Giỏ hàng
mshopkeeper_carts (8 fields)
├── Order info: customer_id, inventory_item_id
├── Pricing: quantity, unit_price, total_price
└── Meta: notes, created_at, updated_at

-- Điểm thành viên
mshopkeeper_customer_points (10 fields)
├── Customer: customer_mshopkeeper_id, name, code
├── Points: total, available, used, expired
└── Sync: last_synced_at, sync_status
```

### 8.2 API Integration Pattern

```php
// Service Layer
MShopKeeperService::class
├── authenticate()          // Xác thực API
├── getCategories()         // Lấy danh mục
├── getInventoryItems()     // Lấy hàng hóa
├── getCustomers()          // Lấy khách hàng
└── getCustomerPoints()     // Lấy điểm thưởng

// Command Layer  
├── SyncMShopKeeperCategories::class
├── SyncMShopKeeperInventoryItems::class
├── SyncMShopKeeperCustomers::class
└── SyncMShopKeeperCustomerPoints::class

// Model Layer (với Scopes & Accessors)
├── MShopKeeperCategory::class
├── MShopKeeperInventoryItem::class  
├── MShopKeeperCustomer::class
└── MShopKeeperCustomerPoint::class
```

### 8.3 Caching Strategy

```php
// Cache keys
'mshopkeeper.categories'           // 30 phút
'mshopkeeper.inventory.{type}'     // 15 phút  
'mshopkeeper.customers'            // 5 phút
'mshopkeeper.customer.points'      // 10 phút

// Cache tags cho invalidation
['mshopkeeper', 'categories']
['mshopkeeper', 'inventory']
['mshopkeeper', 'customers']
```

---

## 🚀 9. DEPLOYMENT & MAINTENANCE

### 9.1 Cron Jobs Setup

```bash
# Trong crontab
# Sync categories mỗi 30 phút
*/30 * * * * php /path/to/artisan mshopkeeper:sync-categories

# Sync inventory mỗi 15 phút  
*/15 * * * * php /path/to/artisan mshopkeeper:sync-inventory-items --include-inventory

# Sync customers mỗi giờ
0 * * * * php /path/to/artisan mshopkeeper:sync-customers

# Sync points mỗi 2 giờ
0 */2 * * * php /path/to/artisan mshopkeeper:sync-customer-points
```

### 9.2 Monitoring & Alerts

```php
// Health checks
- API connectivity
- Sync success rates  
- Database performance
- Cache hit rates
- Error frequencies

// Alerts
- Sync failures > 3 consecutive
- API response time > 5s
- Database queries > 100ms
- Cache miss rate > 50%
```

### 9.3 Backup Strategy

```bash
# Database backup
mysqldump --single-transaction vuphuc_db > backup_$(date +%Y%m%d).sql

# Files backup  
tar -czf storage_backup_$(date +%Y%m%d).tar.gz storage/

# Sync logs backup
cp storage/logs/laravel.log logs_backup_$(date +%Y%m%d).log
```

---

## 📈 10. PERFORMANCE & OPTIMIZATION

### 10.1 Database Optimization

```sql
-- Indexes quan trọng
CREATE INDEX idx_mshopkeeper_id ON mshopkeeper_inventory_items(mshopkeeper_id);
CREATE INDEX idx_category_parent ON mshopkeeper_categories(parent_id);
CREATE INDEX idx_customer_tel ON mshopkeeper_customers(normalized_tel);
CREATE INDEX idx_sync_status ON mshopkeeper_inventory_items(sync_status);
CREATE INDEX idx_is_visible_featured ON mshopkeeper_inventory_items(is_visible, is_featured);
```

### 10.2 Query Optimization

```php
// Eager loading relationships
MShopKeeperInventoryItem::with(['category', 'stocks'])
    ->where('is_visible', true)
    ->where('inactive', false)
    ->get();

// Chunked processing cho sync
MShopKeeperInventoryItem::chunk(100, function ($items) {
    // Process batch
});
```

### 10.3 Frontend Optimization

```javascript
// AJAX với debouncing
const searchProducts = debounce(function(query) {
    // Search API call
}, 300);

// Lazy loading images
<img loading="lazy" src="product-image.jpg" />

// Cache API responses
localStorage.setItem('cart_data', JSON.stringify(cartData));
```

---

## 🔍 11. TROUBLESHOOTING GUIDE

### 11.1 Common Issues

#### Sync Failures

```bash
# Check API connectivity
php artisan mshopkeeper:test-connection

# Check sync status
php artisan mshopkeeper:sync-status

# Force re-sync
php artisan mshopkeeper:sync-categories --force
```

#### Performance Issues

```bash
# Clear all caches
php artisan cache:clear
php artisan config:clear
php artisan view:clear

# Optimize autoloader
composer dump-autoload --optimize

# Database optimization
php artisan db:optimize
```

#### Data Inconsistencies

```bash
# Verify data integrity
php artisan mshopkeeper:verify-data

# Rebuild relationships
php artisan mshopkeeper:rebuild-hierarchy

# Clean orphaned records
php artisan mshopkeeper:cleanup-orphans
```

### 11.2 Debug Commands

```bash
# Enable debug mode
APP_DEBUG=true

# Check logs
tail -f storage/logs/laravel.log

# API debug
MSHOPKEEPER_DEBUG=true php artisan mshopkeeper:sync-categories
```

---

## 📝 12. BEST PRACTICES

### 12.1 Development Guidelines

- ✅ **Luôn test sync commands trước khi deploy**
- ✅ **Backup database trước khi chạy migration**
- ✅ **Sử dụng transactions cho bulk operations**
- ✅ **Implement proper error handling**
- ✅ **Log tất cả API calls và responses**
- ✅ **Cache frequently accessed data**
- ✅ **Validate data trước khi lưu database**

### 12.2 Security Considerations

- ✅ **API credentials trong .env file**
- ✅ **Rate limiting cho API calls**
- ✅ **Input validation và sanitization**
- ✅ **CSRF protection cho forms**
- ✅ **SQL injection prevention**
- ✅ **XSS protection trong views**

### 12.3 Maintenance Tasks

```bash
# Weekly tasks
- Check sync success rates
- Review error logs
- Update API credentials if needed
- Backup database

# Monthly tasks  
- Analyze performance metrics
- Clean up old logs
- Update dependencies
- Review security patches
```

---

## 🎯 13. FUTURE ENHANCEMENTS

### 13.1 Planned Features

- 🔄 **Real-time sync**: WebSocket integration
- 📱 **Mobile API**: REST API cho mobile app
- 🔔 **Push notifications**: Thông báo đơn hàng mới
- 📊 **Advanced analytics**: Dashboard thống kê chi tiết
- 🎨 **Theme customization**: Tùy chỉnh giao diện
- 🌐 **Multi-language**: Hỗ trợ đa ngôn ngữ

### 13.2 Technical Improvements

- ⚡ **Queue system**: Background job processing
- 🔍 **Full-text search**: Elasticsearch integration
- 📈 **Performance monitoring**: APM tools
- 🔐 **OAuth integration**: Social login
- 💳 **Payment gateway**: Online payment
- 📦 **Inventory alerts**: Low stock notifications

---

## 📞 14. SUPPORT & CONTACT

### 14.1 Technical Support

- **Developer**: Vũ Phúc Development Team
- **Documentation**: `/docs` folder
- **Issue Tracking**: Internal ticketing system
- **Emergency Contact**: [Contact Information]

### 14.2 Resources

- **API Documentation**: MShopKeeper API Docs
- **Laravel Documentation**: https://laravel.com/docs
- **Filament Documentation**: https://filamentphp.com/docs

---

*📅 Document created: {{ date('Y-m-d H:i:s') }}*
*🔄 Last updated: {{ date('Y-m-d H:i:s') }}*
*📝 Version: 1.0*
*👨‍💻 Author: Vũ Phúc Development Team*

---

**🎵 Workflow documentation completed successfully!**
