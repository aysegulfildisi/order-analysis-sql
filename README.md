# order-analysis-sql
SQL ile aylık sipariş dağılımı ve temel veri analizi
# order-analysis-sql

Bu projede, SQL kullanılarak e-ticaret verileri üzerinden sipariş analizleri yapılmıştır. Özellikle `order_approved_at` tarihi üzerinden aylık sipariş adetleri hesaplanmıştır.

## 🔍 Sorgu Açıklaması

Aşağıdaki sorgu ile, her ay onaylanan siparişlerin sayısı hesaplanmıştır:

```sql
select date_trunc('month', order_approved_at)::date as month,
       count(order_id) as total_order
from orders
where order_approved_at is not null
group by 1
order by 1 asc;
🗂 Kullanılan Veri Seti
orders, customers, payments, products, reviews, sellers, order_items, translation tabloları

📊 Örnek Çıktı
Veri setinden elde edilen aylık sipariş dağılımı aşağıdaki gibidir:


🔗 İlişkisel Şema
ilişkisel şema.png
