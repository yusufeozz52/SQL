# SQL
SQL kütüphanesi

## 📌 İçindekiler
1. [SQL Sorgularının Çalışma Sırası](#calisma-sirasi)
2. [Veri Çekme ve Filtreleme](#veri-cekme)
3. [Gruplama ve Toplulaştırma](#gruplama)
4. [İlişkisel Sorgular](#iliskisel-sorgular)
5. [Alt Sorgular ve İç İçe Sorgular](#alt-sorgular)
6. [Sık Kullanılan Yerleşik Fonksiyonlar](#fonksiyonlar)
7. [Örneklerde Kullanılan Veritabanı Şeması](#veritabani-semasi)

---

<a name="calisma-sirasi"></a>
## ⚙️ SQL Sorgularının Çalışma Sırası

Hatasız ve performanslı sorgular yazabilmek için yazım sırası ile arka plandaki mantıksal çalışma sırasının farkını bilmek kritik önem taşır.

```text
1. FROM        (Verinin alınacağı tablolar belirlenir)
2. ON          (Birleştirme koşulları uygulanır)
3. JOIN        (Tablolar birleştirilir)
4. WHERE       (Satır bazlı filtreleme yapılır)
5. GROUP BY    (Veriler gruplanır)
6. HAVING      (Gruplanmış veri filtrelenir)
7. SELECT      (Gösterilecek sütunlar seçilir ve hesaplanır)
8. DISTINCT    (Tekrarlayan satırlar temizlenir)
9. ORDER BY    (Sonuçlar sıralanır)
10. LIMIT      (Getirilecek satır sayısı sınırlandırılır)
```

(Not: SELECT sözcüğü sorgunun en başında kullanılsa da,
veritabanı motorları öncelikle FROM ve JOIN komutlarını çalıştırarak kaynak veriyi toplayıp,
WHERE ile de filtreleme yaptığı için listeyi işleme alınma sıralarını dikkate alarak oluşturdum.)

---

<a name="veri-cekme"></a>
## 🔍 Veri Çekme ve Filtreleme

 Temel SELECT ve Mantıksal Koşullar

--> 'İstanbul' veya 'Ankara' şehirlerinde bulunan aktif müşterileri getir
```sql
SELECT customer_id, first_name, last_name, city
FROM customers
WHERE is_active = TRUE 
  AND city IN ('İstanbul', 'Ankara')
ORDER BY last_name ASC;
```
 (Pattern Matching) ve NULL Kontrolü 
 
--> İsmi 'Pro' ile başlayan ve fiyatı tanımlı olan ürünleri bul.
```sql
SELECT product_id, product_name, price
FROM products
WHERE product_name LIKE 'Pro%'
  AND price IS NOT NULL;
```

<a name="gruplama"></a>
## 📊 Gruplama ve Toplulaştırma

WHERE ile HAVING arasındaki temel fark:

 -   WHERE: Gruplama yapılmadan önce satırları filtreler.

 -  HAVING: Gruplama yapıldıktan sonra oluşan özet veriyi (grupları) filtreler.

--> Sadece tamamlanmış siparişler için kategori bazında toplam satışı ve ortalama sipariş tutarını hesapla.

--> Toplam sipariş sayısı 5'ten az olan kategorileri dışarıda bırak.
```sql
SELECT 
    category_id,
    COUNT(order_id) AS total_orders,
    AVG(total_amount) AS avg_order_value,
    SUM(total_amount) AS total_revenue
FROM orders
WHERE order_status = 'COMPLETED'
GROUP BY category_id
HAVING COUNT(order_id) >= 5
ORDER BY total_revenue DESC;
```
---

<a name="iliskisel-sorgular"></a>
## 🔗 İlişkisel Sorgular

 JOIN Türü - Açıklama 

-> INNER JOIN:  Her iki tabloda da yalnızca eşleşen kayıtları getirir.

-> LEFT JOIN:   Sol tablodaki tüm kayıtları ve sağ tabloda eşleşen kayıtları getirir.(Eşleşmeyen sağ taraf NULL olur)

-> RIGHT JOIN:  Sağ tablodaki tüm kayıtları ve sol tabloda eşleşen kayıtları getirir.

---

--> Sipariş detaylarını, müşteri adlarını ve ilgilenen çalışan adlarını birleştirerek getir

```sql
SELECT 
    o.order_id,
    c.first_name AS customer_name,
    e.first_name AS employee_name,
    o.order_date,
    o.total_amount
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
LEFT JOIN employees e ON o.employee_id = e.employee_id;
```
---

<a name="alt-sorgular"></a>
## 🧠 Alt Sorgular ve İç İçe Sorgular

Alt sorgu (Subquery); 'SELECT', 'INSERT', 'UPDATE' veya 'DELETE' komutlarının ya da başka bir alt sorgunun içine yuvalanmış sorgulardır.

1. Tek Değer Döndüren Alt Sorgu (WHERE İfadesinde)

- Genel ortalama maaşın üzerinde kazanan çalışanları bulun:

```sql
SELECT employee_id, first_name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary) 
    FROM employees
)
ORDER BY salary DESC;
```
--- 

2. Çoklu Değer Döndüren Alt Sorgu (IN Operatörü İle)

- En az bir adet yüksek tutarlı sipariş (> 1.000 TL) vermiş müşterileri bulun:

```sql
SELECT customer_id, first_name, last_name
FROM customers
WHERE customer_id IN (
    SELECT DISTINCT customer_id 
    FROM orders 
    WHERE total_amount > 1000
);
```

---

3. İlişkili Alt Sorgu (Correlated Subquery - EXISTS Operatörü İle)

- Hiç sipariş edilmemiş ürünleri sağlayan tedarikçileri listeleyin:

```sql
SELECT supplier_id, company_name
FROM suppliers s
WHERE NOT EXISTS (
    SELECT 1 
    FROM products p
    JOIN order_details od ON p.product_id = od.product_id
    WHERE p.supplier_id = s.supplier_id
);
```
---

4. Türetilmiş Tablo Olarak Alt Sorgu (FROM İfadesinde)

- Departman bazında üretilen toplam gelirin ortalamasını hesaplayın:

```sql
SELECT 
    dept_summary.department_name,
    AVG(dept_summary.total_dept_revenue) AS avg_revenue
FROM (
    SELECT 
        d.department_name,
        SUM(o.total_amount) AS total_dept_revenue
    FROM departments d
    JOIN employees e ON d.department_id = e.department_id
    JOIN orders o ON e.employee_id = o.employee_id
    GROUP BY d.department_name
) AS dept_summary
GROUP BY dept_summary.department_name;
```

---

<a name="fonksiyonlar"></a>
## 🛠️ Sık Kullanılan Yerleşik Fonksiyonlar

Metin İşlemleri (String Manipulation)

    -> CONCAT(str1, str2): İki veya daha fazla metni birleştirir.

    -> SUBSTRING(string, start, length): Belirtilen konumdan metin parçası alır.

    -> UPPER(str) / LOWER(str): Metni büyük veya küçük harfe dönüştürür.

Matematiksel ve Toplulaştırma Fonksiyonları

    -> ROUND(val, decimals): Sayıyı belirtilen hassasiyete göre yuvarlar.

    -> COALESCE(val1, val2, ...): Listedeki ilk NULL olmayan değeri döndürür.

    -> MOD(n, m): n sayısının m sayısına bölümünden kalanı (mod) verir.

  
<a name="veritabani-semasi"></a>
## 🗄️ Örneklerde Kullanılan Veritabanı Şeması

[MÜŞTERİLER] 1 ─── N [SİPARİŞLER] 1 ─── N [SİPARİŞ_DETAYLARI] N ─── 1 [ÜRÜNLER]
                         │
                         N ─── 1 [ÇALIŞANLAR] N ─── 1 [DEPARTMANLAR]


   ---                      


