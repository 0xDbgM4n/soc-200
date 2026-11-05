
**Phân loại:**

|Type|Đặc điểm|
|---|---|
|**In-band (UNION-based)**|Kết quả trả về ngay trong response|
|**Boolean-based blind**|TRUE/FALSE dựa vào response khác nhau|
|**Time-based blind**|Đo response time qua SLEEP()|
**Ví dụ UNION-based:**
```
https://megacorpone.local/tshirts?color=purple'+UNION+SELECT+username,passwords+FROM+administrators--
```

Kết quả SQL:
```
SELECT * FROM tshirts WHERE color = 'purple' 
UNION SELECT username, passwords FROM administrators;
```

**Ví dụ Time-based blind:**
```
https://megacorpone.local/tshirt_model=scoop'+AND+IF(1=1,+sleep(20),+false)
```

### Phát hiện SQL Injection qua Log

**Dấu hiệu trong `access.log`:**
```
192.168.50.50 - - "POST /wp-admin/admin-ajax.php HTTP/1.1" 200 77 "-" "sqlmap/1.5.7#stable (http://sqlmap.org)"
```

- **User-Agent = sqlmap** → lộ ngay tool
    
- Nhiều POST requests lặp lại liên tục → bất thường

**Hạn chế:** Apache log không chứa POST body → cần **ModSecurity** (đã có sẵn trên Ubuntu lab).

**Phân tích ModSec audit log:**
```
sudo cat /var/log/apache2/modsec_audit.log | awk '/-A--/,/-F--/'
```

Kết quả:
```
POST /wp-admin/admin-ajax.php HTTP/1.1
...
action=spAjaxResults&pollid=2%27%29%20AND%201199%3D%28SELECT%20...SLEEP(5)... 
```

**URL-decode** → thấy payload:
```
2') AND 1199=(SELECT 1199 FROM PG_SLEEP(5)) AND ('RdCO'='RdCO
```

**Cảnh báo từ ModSec:**
```
Message: Warning. detected SQLi using libinjection with fingerprint '1)&(1'
[id "942100"] [msg "SQL Injection Attack Detected via libinjection"]
[severity "CRITICAL"]
```

