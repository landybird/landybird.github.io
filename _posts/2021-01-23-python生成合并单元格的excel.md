---
title: 使用openpyxl或者pandas生成excel文件
description: 读取数据生成合并单元格的excel
categories:
- python
tags:
- python
---

#### 原始数据:

```python
data = {
        "ADVERTISING_AND_MARKETING":[
            "CREATIVE_AGENCY",
            "FULL_SERVICE_AGENCY",
            "BUYING_AGENCY",
            "PLANNING_AGENCY",
            "RESELLER"
        ],
        "AGRICULTURE":[
            "FARMING_AND_RANCHING",
            "FISHING_AND_HUNTING_AND_FORESTRY_AND_LOGGING"
        ],
        "AUTOMOTIVE":[
            "T1_MOTORCYCLE",
            "RECREATIONAL",
            "T1_AUTOMOTIVE_MANUFACTURER",
            "T2_DEALER_ASSOCIATIONS",
            "T3_AUTO_AGENCY",
            "T3_AUTO_RESELLERS",
            "T3_DEALER_GROUPS",
            "T3_FRANCHISE_DEALER",
            "T3_INDEPENDENT_DEALER",
            "T3_PARTS_AND_SERVICES",
            "T3_PORTALS"
        ],
        "BANKING_AND_CREDIT_CARDS":[
            "RETAIL_AND_CREDIT_UNION_AND_COMMERCIAL_BANK",
            "ACCOUNTING_AND_TAX",
            "CREDIT_AND_FINANCING_AND_MORTAGES",
            "FIN_TECH",
            "INVESTMENT_BANK_AND_BROKERAGE",
            "PAYMENT_PROCESSING_AND_GATEWAY_SOLUTIONS"
        ],
        "BUSINESS_TO_BUSINESS":[
            "BUSINESS_CONSULTANTS",
            "CALL_CENTER_AND_MESSAGING_SERVICES",
            "COMPUTER_AND_SOFTWARE_AND_HARDWARE",
            "FILE_STORAGE_AND_CLOUD_AND_DATA_SERVICES",
            "HR_AND_FINANCIAL_MANAGEMENT",
            "LOGISTICS_AND_TRANSPORTATION_AND_FLEET_MANAGEMENT",
            "MANUFACTURING",
            "OFFICE_OR_BUSINESS_SUPPLIES",
            "SOFTWARE_AS_A_SERVICE",
            "WEBSITE_DESIGNERS_OR_GRAPHIC_DESIGNERS",
            "WHOLESALE"
        ],
        "CONSUMER_PACKAGED_GOODS":[
            "BEER_AND_WINE_AND_LIQUOR_AND_MALT_BEVERAGES",
            "FOOD",
            "APPAREL_AND_ACCESSORIES",
            "BABY",
            "BEAUTY",
            "HOUSEHOLD_GOODS_DURABLE",
            "HOUSEHOLD_GOODS_NON_DURABLE",
            "OFFICE",
            "PERSONAL_CARE",
            "PET",
            "TOBACCO",
            "VITAMINS_OR_WELLNESS",
            "WATER_AND_SOFT_DRINK_AND_BAVERAGE",
            "HOME_IMPROVEMENT"
        ],
        "ECOMMERCE":[
            "APPAREL_AND_ACCESSORIES",
            "BEAUTY",
            "HOUSEHOLD_GOODS_DURABLE",
            "HOUSEHOLD_GOODS_NON_DURABLE",
            "VIRTUAL_SERVICES",
            "VITAMINS_OR_WELLNESS"
        ],
        "EDUCATION":[
            "ED_TECH",
            "TRADE_SCHOOL",
            "NOT_FOR_PROFIT_COLLEGES_AND_UNIVERSITIES",
            "SCHOOL_AND_EARLY_CHILDREN_EDCATION",
            "EDUCATION_RESOURCES",
            "FOR_PROFIT_COLLEGES_AND_UNIVERSITIES",
            "ELEARNING_AND_MASSIVE_ONLINE_OPEN_COURSES",
            "DIGITAL_NATIVE_EDUCATION_OR_TRAINING"
        ],
        "ENERGY_AND_NATURAL_RESOURCES_AND_UTILITIES":[
            "OIL_AND_GAS_AND_CONSUMABLE_FUEL",
            "MINING_AND_QUARRYING",
            "UTILITIES_AND_ENERGY_EQUIPMENT_AND_SERVICES"
        ],
        "ENTERTAINMENT_AND_MEDIA":[
            "BROADCAST_TELEVISION",
            "ACTIVITIES_AND_LEISURE",
            "AUDIO_STREAMING",
            "CABLE_TELEVISION",
            "EVENTS",
            "MOVIES",
            "MUSEUMS_AND_PARKS_AND_LIBRARIES",
            "MUSIC",
            "PEOPLE",
            "RADIO",
            "SPORTS",
            "TICKETING",
            "VIDEO_STREAMING"
        ],
        "GAMING":[
            "MOBILE_GAMING",
            "AR_OR_VR_GAMING",
            "CONSOLE_AND_CROSS_PLATFORM_GAMING",
            "ESPORTS",
            "PC_GAMING",
            "REAL_MONEY_GAMING"
        ],
        "GOVERNMENT":[
            "ELECTION_COMMISSION",
            "GOVERNMENT_CONTROLLED_ENTITY",
            "GOVERNMENT_DEPARTMENT_OR_AGENCY",
            "GOVERNMENT_OFFICIAL",
            "GOVERNMENT_OWNED_MEDIA",
            "HEAD_OF_STATE",
            "INTERNATIONAL_ORGANIZATON"
        ],
        "HEALTHCARE_AND_PHARMACEUTICALS_AND_BIOTECH":[
            "CLINICAL_TRIALS",
            "COUNSELING_AND_PSYCHOTHERAPY",
            "DIETING_AND_FITNESS_PROGRAMS",
            "HEALTH_SYSTEMS_AND_PRACTITIONERS",
            "HEALTH_TECH",
            "MEDICAL_DEVICES_AND_SUPPLIES_AND_EQUIPMENT",
            "MEDSPA_AND_ELECTIVE_SURGERIES_AND_ALTERNATIVE_MEDICINE",
            "NON_PRESCRIPTION",
            "PRESCRIPTION",
            "RESIDENTIAL_AND_LONG_TERM_CARE_FACILITIES_AND_OUTPATIENT_CARE_CENTERS",
            "VETERINARY_CLINICS_AND_SERVICES",
            "VITAMINS_OR_WELLNESS"
        ],
        "INSURANCE":[
            "AUTO_INSURANCE",
            "HEALTH_INSURANCE",
            "HOME_INSURANCE",
            "INSURANCE_TECH",
            "LIFE_INSURANCE",
            "PROPERTY_AND_CASUALTY"
        ],
        "NON_PROFIT":[
            "CHRONIC_CONDITIONS_AND_MEDICAL_CAUSES",
            "ENVIRONMENT_AND_ANIMAL_WELFARE",
            "HUMANITARIAN_OR_DISASTER_RELIEF",
            "RELIGIOUS"
        ],
        "ORGANIZATIONS_AND_ASSOCIATIONS":[
            "ARTS_AND_HERITAGE_AND_EDUCATION",
            "CIVIC_INFLUENCERS",
            "ISSUE_ADVOCACY",
            "PROFESSIONAL_ASSOCIATIONS",
            "SPORTING_AND_OUTDOOR",
            "RELIGIOUS"
        ],
        "PROFESSIONAL_SERVICES":[
            "CAREER",
            "AUTO",
            "CONSULTING",
            "ENGINEERING_AND_DESIGN",
            "FITNESS",
            "LEGAL",
            "PACKAGE_OR_FREIGHT_DELIVERY",
            "PHOTOGRAPHY_AND_FILMING_SERVICES",
            "REAL_ESTATE",
            "SAFETY_SERVICES",
            "WAREHOUSING_AND_STORAGE",
            "PERSONAL_CARE"
        ],
        "POLITICS":[
            "CANDIDATE_OR_POLITICIAN",
            "INDEPENDENT_EXPENDITURE_GROUP",
            "PARTY_INDEPENDENT_EXPENDITURE_GROUP_US",
            "BALLOT_INITIATIVE_OR_REFERENDUM",
            "POLITICAL_PARTY_OR_COMMITTEE"
        ],
        "PUBLISHING":[
            "BEAUTY_AND_FASHION",
            "CAREER_AND_TECH",
            "CULTURE_AND_LIFESTYLE",
            "FINANCE",
            "FOOD",
            "NEWS_AND_CURRENT_EVENTS",
            "ONLINE_ONLY_PUBLICATIONS",
            "SCHOLARLY"
        ],
        "RESTAURANTS":[
            "CASUAL_DINING",
            "COFFEE",
            "DRINKING_PLACES",
            "PIZZA",
            "QUICK_SERVICE"
        ],
        "RETAIL":[
            "BOOKSTORES",
            "ELECTRONICS_AND_APPLIANCES",
            "DEPARTMENT_STORE",
            "GROCERY_AND_DRUG_AND_CONVENIENCE",
            "FOOTWEAR",
            "HOME_AND_FURNITURE_AND_OFFICE",
            "SPORTING",
            "SUPERSTORES",
            "TOY_AND_HOBBY",
            "PET",
            "HOME_IMPROVEMENT",
            "BEAUTY",
            "APPAREL_AND_ACCESSORIES",
            "BEER_AND_WINE_AND_LIQUOR_AND_MALT_BEVERAGES"
        ],
        "TECHNOLOGY":[
            "CONSUMER_ELECTRONICS",
            "DATA_ANALYTICS_AND_DATA_MANAGEMENT",
            "DATING_AND_TECHNOLOGY_APPS",
            "DESKTOP_SOFTWARE",
            "HOME_TECH",
            "NETWORK_SECURITY_PRODUCTS",
            "SOCIAL_MEDIA"
        ],
        "TELECOM":[
            "WIRELESS_SERVICES",
            "CABLE_AND_SATELLITE",
            "TELECOMMUNICATIONS_EQUIPMENT_AND_ACCESSORIES",
            "TELEPHONE_SERVICE_PROVIDERS_AND_CARRIERS"
        ],
        "TRAVEL":[
            "RAILROADS",
            "HOTEL_AND_ACCOMODATION",
            "RIDE_SHARING_OR_TAXI_SERVICES",
            "AUTO_RENTAL",
            "TOURISM_AND_TRAVEL_SERVICES",
            "TOURISM_BOARD",
            "AIR",
            "TRAVEL_AGENCIES_AND_GUIDES_AND_OTAS",
            "CRUISES_AND_MARINE"
        ]
    }
```

#### 使用`openpyxl`

```python

import pandas as pd
from openpyxl import workbook
from openpyxl.styles import Color, Alignment, Font, PatternFill

header = ["vertical", "subvertical"]
wb = workbook.Workbook()
ws = wb.active
ws.append(header)
ws.column_dimensions["A"].width = 80
ws.column_dimensions["B"].width = 80

common_background = "e4e5e8"
common_fill = PatternFill(start_color=common_background, end_color=common_background, fill_type="solid")
ft_title = Font(name='微软雅黑', color='0e0b0d', size=16, bold=True)
ft = Font(name='微软雅黑', color='0e0b0d', size=12)
ali = Alignment(horizontal='center', vertical='center')

a1, b1 = ws["A1"],ws["B1"]
a1.alignment = b1.alignment = ali
a1.font = b1.font = ft_title
a1.fill = b1.fill = common_fill

merge_row = "A{}:A{}"
total_row = "A2:B{}"

merge_count_start = 2
merge_count_end = 1

for k, values in data.items():
    for v in values:
        ws.append([k, v])
        merge_count_end += 1
    merge_row_ = merge_row.format(merge_count_start, merge_count_end)
    ws.merge_cells(merge_row_)
    merge_count_start = merge_count_end + 1

for cells in ws[total_row.format(merge_count_end)]:
    for cell in cells:
        cell.alignment = ali
        cell.font = ft

wb.save("demo.xlsx")

```

#### 使用pandas

```python
df = pd.json_normalize(data)
#        ADVERTISING_AND_MARKETING  ...                                         AUTOMOTIVE
# 0  [CREATIVE_AGENCY, FULL_SERVICE_AGENCY, BUYING_...  ...  [T1_MOTORCYCLE, RECREATIONAL, T1_AUTOMOTIVE_MA...
df = df.transpose()
# ADVERTISING_AND_MARKETING  [CREATIVE_AGENCY, FULL_SERVICE_AGENCY, BUYING_...
# AGRICULTURE                [FARMING_AND_RANCHING, FISHING_AND_HUNTING_AND...
# AUTOMOTIVE                 [T1_MOTORCYCLE, RECREATIONAL, T1_AUTOMOTIVE_MA...
df = df.explode(0)
#                                                                       0
# ADVERTISING_AND_MARKETING                               CREATIVE_AGENCY
# ADVERTISING_AND_MARKETING                           FULL_SERVICE_AGENCY
# ADVERTISING_AND_MARKETING                                 BUYING_AGENCY
# AGRICULTURE                                        FARMING_AND_RANCHING
# AGRICULTURE                FISHING_AND_HUNTING_AND_FORESTRY_AND_LOGGING
# AUTOMOTIVE                                                T1_MOTORCYCLE
df = df.reset_index()
#                        index                                             0
# 0  ADVERTISING_AND_MARKETING                               CREATIVE_AGENCY
# 1  ADVERTISING_AND_MARKETING                           FULL_SERVICE_AGENCY
# 2  ADVERTISING_AND_MARKETING                                 BUYING_AGENCY
# 3                AGRICULTURE                          FARMING_AND_RANCHING
# 4                AGRICULTURE  FISHING_AND_HUNTING_AND_FORESTRY_AND_LOGGING
# 5                 AUTOMOTIVE                                 T1_MOTORCYCLE
df = df.rename(columns={'index': 'vertical', 0 :'subvertical'})
#                     vertical                                   subvertical
# 0  ADVERTISING_AND_MARKETING                               CREATIVE_AGENCY
# 1  ADVERTISING_AND_MARKETING                           FULL_SERVICE_AGENCY
# 2  ADVERTISING_AND_MARKETING                                 BUYING_AGENCY
# 3                AGRICULTURE                          FARMING_AND_RANCHING
# 4                AGRICULTURE  FISHING_AND_HUNTING_AND_FORESTRY_AND_LOGGING
# 5                 AUTOMOTIVE                                 T1_MOTORCYCLE

df.loc[df['vertical'].duplicated(), 'vertical'] = ''
 # Find all duplicates in the column and replace them with empty strings
df.to_excel('out.xlsx', index=False)
```
