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
[ilişkisel şema.png](https://github.com/aysegulfildisi/order-analysis-sql/blob/main/ilis%CC%A7kisel%20s%CC%A7ema.png)

case1:
Question 1 : Aylık olarak order dağılımını inceleyiniz. Tarih verisi için order_approved_at kullanılmalıdır.
select (date_trunc('month', order_approved_at))::date as month,
count (order_id) as total_order
from orders
where order_approved_at is not NULL
group by 1
order by 1 asc

Question 2 : Aylık olarak order status kırılımında order sayılarını inceleyiniz. Sorgu sonucunda çıkan outputu excel ile görselleştiriniz. Dramatik bir düşüşün ya da yükselişin olduğu aylar var mı? Veriyi inceleyerek yorumlayınız.

select(date_trunc('month', order_approved_at))::date as month, order_status,
count (order_id) as total_order
from orders
where order_approved_at is not NULL
group by 1,2
order by 2 asc

Question 3 : Ürün kategorisi kırılımında sipariş sayılarını inceleyiniz. Özel günlerde öne çıkan kategoriler nelerdir? Örneğin yılbaşı, sevgililer günü…

with order_cat as( 
	select 
	to_char(date_trunc('month', order_approved_at), 'YYYY-MM') as month,
	o.order_id,p.product_category_name,t.category_name_english
	from orders as o
	join order_items as oi on o.order_id=oi.order_id
	join products as p on oi.product_id=p.product_id
	join translation as t on t.category_name=p.product_category_name
	where order_approved_at is not NULL
	)
select oc.category_name_english,oc.month,
count ( distinct oc.order_id) as order_count
from order_cat as oc
where oc.product_category_name is not null and oc.month is not null
group by 1,2
order by 3 desc

Question 4 :Haftanın günleri(pazartesi, perşembe, ….) ve ay günleri (ayın 1’i,2’si gibi) bazında order sayılarını inceleyiniz. Yazdığınız sorgunun outputu ile excel’de bir görsel oluşturup yorumlayınız.

(select
to_char(order_approved_at, 'day') as "days of week",
count(order_id) as order_count
from orders
where to_char(order_approved_at, 'day') is not null
group by 1
order by 2)
union
(select
to_char(order_approved_at, 'dd') as days_of_month,
count(order_id) as order_count
from orders
where to_char(order_approved_at, 'dd') is not null
group by 1
order by 2)
<img width="470" height="322" alt="image" src="https://github.com/user-attachments/assets/6fc20582-7442-4096-b94f-faf738d60e0f" />

