<img width="470" height="465" alt="image" src="https://github.com/user-attachments/assets/40226848-a18f-403a-b1f7-f1fe634323fd" /># order-analysis-sql
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


Case 2 : Müşteri Analizi 
Question 1 : Hangi şehirlerdeki müşteriler daha çok alışveriş yapıyor? Müşterinin şehrini en çok sipariş verdiği şehir olarak belirleyip analizi ona göre yapınız. 

Örneğin; Sibel Çanakkale’den 3, Muğla’dan 8 ve İstanbul’dan 10 sipariş olmak üzere 3 farklı şehirden sipariş veriyor. Sibel’in şehrini en çok sipariş verdiği şehir olan İstanbul olarak seçmelisiniz ve Sibel’in yaptığı siparişleri İstanbul’dan 21 sipariş vermiş şekilde görünmelidir.

with order_counts as (
    select o.customer_id,customer_city,
    count(distinct order_id) as order_count
    from orders as o
    join customers as c on c.customer_id = o.customer_id
    group by 1,2
       ),   
    customer_city_rn as (
    select row_number() over (partition by customer_id order by order_count desc) as rn,
    customer_id,customer_city
    from order_counts
       ),
   customer_city as (
   select customer_id,customer_city
   from customer_city_rn
    where rn = 1
       ) 
select cc.customer_city,
count(o.order_id)
from orders as o
join customer_city as cc on o.customer_id = cc.customer_id
group by 1
order by 2 desc

Case 3: Satıcı Analizi
Question 1 :Siparişleri en hızlı şekilde müşterilere ulaştıran satıcılar kimlerdir? Top 5 getiriniz. Bu satıcıların order sayıları ile ürünlerindeki yorumlar ve puanlamaları inceleyiniz ve yorumlayınız.

with top5 as(
	select
	avg(order_delivered_customer_date - order_purchase_timestamp) as date_dif,
	Round(avg(review_score), 1) as review_score_average,
	sum(payment_value) as payment_total,
	count (distinct o.order_id) as order_count,
	s.seller_id,
	orr.review_comment_message
	from orders as o
	inner join order_items as oi on o.order_id = oi.order_id
	inner join sellers as s on oi.seller_id= s.seller_id
	inner join reviews as orr on o.order_id = orr.order_id
	inner join payments as op on o.order_id = op.order_id
    where (order_delivered_customer_date::date - order_purchase_timestamp::date)::integer is not null
    group by 5,6
)
select * from top5
where order_count>5 
order by 1 asc
limit 5

Question 2 : Hangi satıcılar daha fazla kategoriye ait ürün satışı yapmaktadır? 
 Fazla kategoriye sahip satıcıların order sayıları da fazla mı? 

with satici as(
	select s.seller_id,
	p.product_category_name,
	count( distinct order_id) as order_count,
	sum(price) as total_price
	from sellers as s
	inner join order_items as oi on s.seller_id=oi.seller_id                        
	inner join products as p on p.product_id =oi.product_id                          
	group by 1,2)
select 
seller_id,
count(product_category_name) as product_category_count,
sum(order_count) as total_order_count,
sum(total_price)::integer as total_sales
from satici
group by 1
order by 2 desc, 3 desc, 4 desc

Case 4 : Payment Analizi
Question 1 : Ödeme yaparken taksit sayısı fazla olan kullanıcılar en çok hangi bölgede yaşamaktadır? Bu çıktıyı yorumlayınız.

select 
count(customer_unique_id) as customer_count,
customer_state,
payment_installments
from customers as c
inner join orders as o on c.customer_id=o.customer_id         
inner join payments as p on p.order_id=o.order_id         
group by 2,3                                     
order by 3 desc, 1 desc
Question 2 : Ödeme tipine göre başarılı order sayısı ve toplam başarılı ödeme tutarını hesaplayınız. En çok kullanılan ödeme tipinden en az olana göre sıralayınız.

select p.payment_type,
count(distinct o.order_id) as order_count,
sum(p.payment_value)::integer as total_payment
from orders as o
join payments as p on o.order_id=p.order_id         
where  o.order_status not in ('cancelled','unavailable')
group by 1
order by 2 desc
Question 3 : Tek çekimde ve taksitle ödenen siparişlerin kategori bazlı analizini yapınız. En çok hangi kategorilerde taksitle ödeme kullanılmaktadır?


-- Sorgunun ilk kısmında tamamlanan siparişler (delivered) üzerinden yapılan taksitlerin hangi ürün çeşitlerinde olduğunu bulduk. Daha sonra tek çekimde ve taksitle ödenen siparişlerin kategori bazlı analizini yaptık.

with order_payments as (
    select
    o.order_id,
    p.payment_type,
    p.payment_installments, 
    pr.product_category_name,
    t.category_name_english
    from orders o
    join payments as p on o.order_id = p.order_id
    join order_items as oi on o.order_id = oi.order_id
    join products as pr on oi.product_id = pr.product_id
    join translation as t on t.category_name=pr.product_category_name
    where o.order_status = 'delivered'
),
taksitle as(
select  count(distinct o.order_id) as order_count_taksitle_cekim,
	p.product_category_name as taksitlicategory_name
	from orders as o
	left join order_items as oi on oi.order_id = o.order_id
	left join products as p on oi.product_id = p.product_id
	left join payments as py on py.order_id = o.order_id
left join translation as t on t.category_name=p.product_category_name
where py.payment_installments > 1
group by 2
order by 1 desc
),
tek_cekim as (
	select  count(distinct o.order_id) as order_count_tek_cekim,
	p.product_category_name as tcekimcategory_name
	from orders as o
	left join order_items as oi on oi.order_id = o.order_id
	left join products as p on oi.product_id = p.product_id
	left join payments as py on py.order_id = o.order_id
left join translation as t on t.category_name=p.product_category_name
where py.payment_installments = 1
group by 2
order by 1 desc
)
select t.taksitlicategory_name,order_count_taksitle_cekim,tc.tcekimcategory_name,order_count_tek_cekim
from taksitle t 
join tek_cekim tc on t.taksitlicategory_name=tc.tcekimcategory_name
Case 5 : RFM Analizi

Aşağıdaki e_commerce_data_.csv doyasındaki veri setini kullanarak RFM analizi yapınız. 
Recency hesaplarken bugünün tarihi değil en son sipariş tarihini baz alınız. 

- - Veri manipülasyonu olarak invoiceno, quantity ve unitprice sıfırdan büyük kabul edilip, C ile başlayan yani iptal edilmiş invoiceno verileri RFM ye dahil edilmemiştir. Bu kısım with ile oluşturulan ilk tabloda tablo1 ile gösterilmiştir.

with tablo1 as (
	select 
	customer_id,invoiceno,quantity,unitprice,
	max(invoicedate) as max_invoicedate
	from rfm 
	where invoiceno not like ‘%C’ and quantity>=0 and unitprice>=0 
           and invoiceno is not null 
            and customer_id is not null
	group by 1,2,3,4
), 
recency as (
	select
	customer_id,
	( '2011-12-09'-max_invoicedate::date) as recency
	from tablo1
),
frequency as (
	select
	customer_id,
	count(distinct invoiceno) as frequency
	from tablo1
	group by 1
),
monetary as (
	select 
	customer_id,
	sum(quantity*unitprice)::integer as monetary
	from tablo1
	group by 1
),
scores as (
	select 
		r.customer_id,
		r.recency,
		ntile(5) over(order by recency desc ) as recency_score, 
		f.frequency,
		ntile(5) over(order by frequency asc ) as frequency_score,
		m.monetary,
		ntile(5) over(order by monetary  asc) as monetary_score
	from recency r
	join frequency f on r.customer_id = f.customer_id
	join monetary m on r.customer_id = m.customer_id
),



Bu aşamaya kadar recency, frequency, monetary değerleri hesaplanmış ve skorlar belirlenmiştir.


