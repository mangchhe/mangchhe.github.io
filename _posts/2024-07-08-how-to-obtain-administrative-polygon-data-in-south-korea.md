---
layout: post
title: How to Obtain Administrative Polygon Data in South Korea
categories: domain
tags: map
---

**행정 구역별 지리 정보**

- [국토교통부](https://www.vworld.kr/dtmk/dtmk_ntads_s002.do?searchKeyword=읍면동&searchOrganization=&searchBrmCode=&searchTagList=&searchFrm=&pageIndex=1&gidmCd=&gidsCd=&sortType=00&svcCde=MK&dsId=30603&listPageIndex=1)
- [지오서비스](https://www.geoservice.co.kr/)
- [공공데이터포털](https://www.data.go.kr/data/59216620/linkedData.do)

**사전 지식**

> [.shp 확장자](https://docs.fileformat.com/ko/gis/shp/)
>
> ESRI Shapefile의 표현에 사용되는 기본 파일 유형 중 하나의 파일 확장자
>
> 데이터 세트의 지리 공간 정보를 벡터 기능 (포인트들, 라인, 폴리곤)
>
> 해당 파일에 포함된 정보를 이해하기 위해 shapefile은 다음과 같은 필수 파일이 사용된다.
>
> - shx 파일 - 인덱스 파일
> - dbf 파일 - 모양의 모든 속성을 기본 파일에 저장하는 dBASE 파일
> - prj 파일 - 파일의 프로젝트 정보 저장


**엑셀 파일 추출**

```python
import geopandas as gpd
import pandas as pd

shapefile_path = "{filepath}/{filename}.shp"
excel_path = "{filepath}/{filename}.xlsx"

gdf = gpd.read_file(shapefile_path, encoding='euc-kr')

df = pd.DataFrame(gdf.drop(columns='geometry'))
df['geometry'] = gdf.geometry.apply(lambda x: x.wkt)

df.to_excel(excel_path, index=False)

print(f"Shapefile data has been exported to {excel_path}")
```

**특정 행정구역 지도 시각화**

```python
import geopandas as gpd
import matplotlib.pyplot as plt
import pandas as pd
from pyproj import Transformer

shapefile_path = "{filepath}/{filename}.shp"

gdf = gpd.read_file(shapefile_path, encoding='euc-kr')

fig, ax = plt.subplots(1, 1, figsize=(10, 10))
gdf.plot(ax=ax, color='lightgrey')

search_value = input()
filtered_gdf = gdf[gdf['EMD_NM'] == search_value]

if filtered_gdf.empty:
    print(f"No results found for SIG_KOR_NM = {search_value}")
else:
    centroid = filtered_gdf.geometry.centroid.iloc[0]
    center_x, center_y = centroid.x, centroid.y
    print(f"Center coordinates (original): ({center_x}, {center_y})")

    transformer = Transformer.from_crs(gdf.crs, "EPSG:4326", always_xy=True)
    lon, lat = transformer.transform(center_x, center_y)
    print(f"Center coordinates (latitude, longitude): ({lat}, {lon})")

    fig, ax = plt.subplots(1, 1, figsize=(10, 10))
    gdf.plot(ax=ax, color='lightgrey')

    filtered_gdf.plot(ax=ax, color='red')
    plt.scatter([center_x], [center_y], color='blue', marker='x', s=100)

    bounds = filtered_gdf.total_bounds
    x_margin = (bounds[2] - bounds[0]) * 3
    y_margin = (bounds[3] - bounds[1]) * 3
    ax.set_xlim([bounds[0] - x_margin, bounds[2] + x_margin])
    ax.set_ylim([bounds[1] - y_margin, bounds[3] + y_margin])
    
    plt.title(f"Map of {search_value}")
    plt.show()
```
