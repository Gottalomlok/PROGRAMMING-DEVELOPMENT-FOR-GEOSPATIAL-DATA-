# Lab 1: การวิเคราะห์พื้นที่ให้บริการของสถานีขนส่งสาธารณะด้วย GeoPandas (ฉบับรายงานทางการ)

---

## 1. บทนำ

การวิเคราะห์ข้อมูลเชิงพื้นที่เป็นองค์ประกอบสำคัญของงานระบบสารสนเทศภูมิศาสตร์ (GIS) โดยเฉพาะในการวางแผนและประเมินโครงสร้างพื้นฐานด้านการคมนาคมขนส่ง ห้องปฏิบัติการนี้มีวัตถุประสงค์เพื่อศึกษาการประยุกต์ใช้เครื่องมือ GeoPandas สำหรับการวิเคราะห์พื้นที่ให้บริการของสถานีขนส่งสาธารณะ ผ่านกระบวนการสร้าง Buffer และ Spatial Join เพื่อประเมินการครอบคลุมพื้นที่เมือง

การปฏิบัติการครั้งนี้ช่วยพัฒนาทักษะการจัดการข้อมูลเชิงพื้นที่ การแปลงระบบพิกัด การคำนวณพื้นที่ให้บริการ และการสื่อสารผลลัพธ์ผ่านแผนที่แบบ Interactive ซึ่งเป็นพื้นฐานสำคัญของการวิเคราะห์เชิงพื้นที่ในงาน GIS สมัยใหม่

---

## 2. ข้อมูลที่ใช้ในการศึกษา

การทดลองใช้ข้อมูลตำแหน่งสถานีขนส่งสาธารณะในเขตกรุงเทพมหานคร ซึ่งดึงมาจาก OpenStreetMap ผ่านไลบรารี osmnx โดยข้อมูลอยู่ในรูปแบบ Point Geometry และถูกนำมาวิเคราะห์ร่วมกับขอบเขตพื้นที่เมือง

---

## 3. ขั้นตอนการดำเนินงาน (Workflow)

### ขั้นตอนที่ 1: ติดตั้งไลบรารี
```python
!pip install geopandas folium osmnx
```
**คำอธิบาย:** ติดตั้งเครื่องมือสำหรับจัดการข้อมูล GIS และสร้างแผนที่

---

### ขั้นตอนที่ 2: นำเข้าไลบรารี
```python
import geopandas as gpd
import folium
import osmnx as ox
```
**คำอธิบาย:** โหลดไลบรารีที่ใช้วิเคราะห์และแสดงผล

---

### ขั้นตอนที่ 3: โหลดข้อมูลสถานีขนส่ง (กรุงเทพมหานคร)
```python
place = "Bangkok, Thailand"
tags = {"amenity": "bus_station"}

stations = ox.features_from_place(place, tags)
stations = stations.reset_index()
stations = gpd.GeoDataFrame(stations, geometry="geometry", crs="EPSG:4326")
```
**คำอธิบาย:** ดึงข้อมูลสถานีขนส่งจาก OpenStreetMap และกำหนดระบบพิกัด

---

### ขั้นตอนที่ 4: แปลงระบบพิกัดและสร้าง Buffer
```python
stations_proj = stations.to_crs(epsg=32647)
stations_proj["buffer"] = stations_proj.geometry.buffer(5000)
buffer_gdf = stations_proj.set_geometry("buffer")
```
**คำอธิบาย:** แปลง CRS เป็นหน่วยเมตรเพื่อสร้าง Buffer ระยะ 5 กิโลเมตร

---

### ขั้นตอนที่ 5: วิเคราะห์พื้นที่ครอบคลุม (Spatial Join ตัวอย่าง)
```python
boundary = stations_proj.buffer(0).unary_union.convex_hull
boundary_gdf = gpd.GeoDataFrame(geometry=[boundary], crs=stations_proj.crs)

join = gpd.sjoin(stations_proj, boundary_gdf, how="left")
```
**คำอธิบาย:** ตรวจสอบความสัมพันธ์เชิงพื้นที่ของข้อมูล

---

### ขั้นตอนที่ 6: สร้าง Interactive Map
```python
stations_wgs = stations_proj.to_crs(epsg=4326)
buffer_wgs = buffer_gdf.to_crs(epsg=4326)

m = folium.Map(location=[13.75, 100.5], zoom_start=10)
folium.GeoJson(buffer_wgs).add_to(m)
folium.GeoJson(stations_wgs).add_to(m)

m
```
**คำอธิบาย:** แสดงพื้นที่บริการและตำแหน่งสถานีในแผนที่แบบโต้ตอบ

---

## 4. ตัวอย่าง Overpass Turbo Query (กรุงเทพมหานคร)

```
[out:json][timeout:25];
area[name="Bangkok"]->.searchArea;
(
  node["amenity"="bus_station"](area.searchArea);
);
out geom;
```

สามารถ Export เป็น GeoJSON เพื่อนำเข้าใช้งานต่อได้

---

## 5. Flow การเขียน Markdown ใน Notebook

```
# Lab Title
## Introduction
## Data Preparation
## Buffer Analysis
## Spatial Join
## Interactive Map
## Discussion / Summary
## Answers to Questions
```

โครงสร้างนี้ช่วยให้รายงานอ่านง่ายและเป็นลำดับขั้นตอน

---

## 6. สรุปผลการเรียนรู้

จากการทดลองนี้ พบว่าสามารถใช้ GeoPandas วิเคราะห์พื้นที่บริการของสถานีขนส่งได้อย่างมีประสิทธิภาพ โดยการสร้าง Buffer และ Spatial Join ช่วยให้เข้าใจการกระจายตัวและการครอบคลุมพื้นที่เมือง พร้อมทั้งสามารถสื่อสารผลลัพธ์ผ่านแผนที่ Interactive ได้อย่างชัดเจน

---

## 7. คำตอบคำถามท้าย Lab

**1. Spatial Join vs Attribute Join**  
Spatial Join ใช้ตำแหน่งทางภูมิศาสตร์ ส่วน Attribute Join ใช้ค่าฟิลด์

**2. เหตุผลที่ต้องแปลง CRS**  
เพื่อให้การคำนวณระยะทางถูกต้องในหน่วยเมตร

**3. เปลี่ยนเป็น 10 กิโลเมตร**  
แก้ buffer เป็น 10000

**4. วิธี Interactive Map ที่เหมาะสม**  
Folium เพราะสามารถโต้ตอบและซูมได้

**5. Buffer ผิดขนาด**  
เกิดจาก CRS ไม่ถูกต้อง → ต้องแปลง CRS ก่อน

---

## 8. การตรวจสอบก่อนส่งงาน

- โหลดข้อมูลได้ถูกต้อง
- CRS ถูกต้องก่อน buffer
- แสดงแผนที่ได้
- โค้ดรันครบทุกเซลล์
- Markdown เป็นลำดับ
- ตอบคำถามครบ

---

## 9. โครงไฟล์ส่งงาน

```
Lab_1_Name_ID.ipynb
│
├── Markdown: Introduction
├── Code: Data Loading
├── Code: Buffer
├── Code: Spatial Join
├── Code: Map
├── Markdown: Summary
└── Markdown: Answers
```

---

เอกสารนี้พร้อมใช้งานสำหรับการส่งรายงาน และสามารถแก้ไขเพิ่มเติมตามข้อกำหนดของผู้สอนได้

