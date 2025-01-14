# Soil chemical and physical properties

::: {.rmdnote}
You are reading the work-in-progress An Open Compendium of Soil Sample and Soil Profile Datasets. This chapter is currently draft version, a peer-review publication is pending. You can find the polished first edition at <https://opengeohub.github.io/SoilSamples/>.
:::

Last update:  2023-05-10



## Overview

This section describes import steps used to produce a global compilation of soil 
laboratory data with chemical (and physical) soil properties that can be then 
used for predictive soil mapping / modeling at global and regional scales.

Read more about soil chemical properties, global soil profile and sample data sets and functionality:

- Arrouays, D., Leenaars, J. G., Richer-de-Forges, A. C., Adhikari, K., Ballabio, C., Greve, M., ... & Heuvelink, G. (2017). [Soil legacy data rescue via GlobalSoilMap and other international and national initiatives](https://doi.org/10.1016/j.grj.2017.06.001). GeoResJ, 14, 1-19.
- Batjes, N. H., Ribeiro, E., van Oostrum, A., Leenaars, J., Hengl, T., & de Jesus, J. M. (2017). [WoSIS: providing standardised soil profile data for the world](http://www.earth-syst-sci-data.net/9/1/2017/). Earth System Science Data, 9(1), 1.
- Hengl, T., MacMillan, R.A., (2019). [Predictive Soil Mapping with R](https://soilmapper.org/). OpenGeoHub foundation, Wageningen, the Netherlands, 370 pages, www.soilmapper.org, ISBN: 978-0-359-30635-0.
- Rossiter, D.G.,: [Compendium of Soil Geographical Databases](https://www.isric.org/explore/soil-geographic-databases).

## ![alt text](./tex/R_logo.svg.png "Packages in use") Specifications

#### Data standards



- Metadata information: ["Soil Survey Investigation Report No. 42."](https://www.nrcs.usda.gov/Internet/FSE_DOCUMENTS/stelprdb1253872.pdf) and ["Soil Survey Investigation Report No. 45."](https://www.nrcs.usda.gov/Internet/FSE_DOCUMENTS/nrcs142p2_052226.pdf),
- Model DB: [National Cooperative Soil Survey (NCSS) Soil Characterization Database](https://ncsslabdatamart.sc.egov.usda.gov/),

#### _Target variables:_


```r
site.names = c("site_key", "usiteid", "site_obsdate", "longitude_decimal_degrees", 
               "latitude_decimal_degrees")
hor.names = c("labsampnum","site_key","layer_sequence","hzn_top","hzn_bot", 
              "hzn_desgn", "tex_psda", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", 
              "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "ph_cacl2", 
              "cec_sum", "cec_nh4", "ecec", "wpg2", "db_od", "ca_ext", "mg_ext", 
              "na_ext", "k_ext", "ec_satp", "ec_12pre")
## target structure:
col.names = c("site_key", "usiteid", "site_obsdate", "longitude_decimal_degrees", 
              "latitude_decimal_degrees", "labsampnum", "layer_sequence", "hzn_top", 
              "hzn_bot","hzn_desgn", "tex_psda", "clay_tot_psa", "silt_tot_psa", 
              "sand_tot_psa", "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", 
              "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "wpg2", "db_od", "ca_ext", 
              "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre", "source_db", 
              "confidence_degree", "project_url", "citation_url")
```

Targe variables listed:

- `clay_tot_psa`: Clay, Total in % wt for <2 mm soil fraction, 
- `silt_tot_psa`: Silt, Total in % wt for <2 mm soil fraction, 
- `sand_tot_psa`: Sand, Total in % wt for <2 mm soil fraction, 
- `oc`: Carbon, Organic in g/kg for <2 mm soil fraction, 
- `oc_d`: Soil organic carbon density in kg/m3, 
- `c_tot`: Carbon, Total in g/kg for <2 mm soil fraction, 
- `n_tot`: Nitrogen, Total NCS in g/kg for <2 mm soil fraction, 
- `ph_kcl`: pH, KCl Suspension for <2 mm soil fraction,
- `ph_h2o`: pH, 1:1 Soil-Water Suspension for <2 mm soil fraction, 
- `ph_cacl2`: pH, CaCl2 Suspension for <2 mm soil fraction, 
- `cec_sum`: Cation Exchange Capacity, Summary, in cmol(+)/kg for <2 mm soil fraction, 
- `cec_nh4`: Cation Exchange Capacity, NH4 prep, in cmol(+)/kg for <2 mm soil fraction, 
- `ecec`: Cation Exchange Capacity, Effective, CMS derived value default, standa prep in cmol(+)/kg for <2 mm soil fraction, 
- `wpg2`: Coarse fragments in % wt for >2 mm soil fraction, 
- `db_od`: Bulk density (Oven Dry) in g/cm3 (4A1h), 
- `ca_ext`: Calcium, Extractable in mg/kg for <2 mm soil fraction (usually Mehlich3), 
- `mg_ext`: Magnesium, Extractable in mg/kg for <2 mm soil fraction (usually Mehlich3), 
- `na_ext`: Sodium, Extractable in mg/kg for <2 mm soil fraction (usually Mehlich3), 
- `k_ext`: Potassium, Extractable in mg/kg for <2 mm soil fraction (usually Mehlich3), 
- `ec_satp`: Electrical Conductivity, Saturation Extract in dS/m for <2 mm soil fraction, 
- `ec_12pre`: Electrical Conductivity,  Predict, 1:2 (w/w) in dS/m for <2 mm soil fraction,

## ![alt text](./tex/R_logo.svg.png "Data import") Data import

#### National Cooperative Soil Survey Characterization Database

- National Cooperative Soil Survey, (2020). National Cooperative Soil Survey Characterization Database. Data download URL: http://ncsslabdatamart.sc.egov.usda.gov/  
- O'Geen, A., Walkinshaw, M., & Beaudette, D. (2017). SoilWeb: A multifaceted interface to soil survey information. Soil Science Society of America Journal, 81(4), 853-862. https://doi.org/10.2136/sssaj2016.11.0386n  

This data set is continuously updated.


```r
if(!exists("chemsprops.NCSS")){
  ncss.site <- read.csv("/mnt/diskstation/data/Soil_points/INT/USDA_NCSS/NCSS_Site_Location.csv", stringsAsFactors = FALSE)
  ncss.layer <- read.csv("/mnt/diskstation/data/Soil_points/INT/USDA_NCSS/NCSS_Layer.csv", stringsAsFactors = FALSE)
  ncss.bdm <- read.csv("/mnt/diskstation/data/Soil_points/INT/USDA_NCSS/NCSS_Bulk_Density_and_Moisture.csv", stringsAsFactors = FALSE)
  ## multiple measurements
  summary(as.factor(ncss.bdm$prep_code))
  ncss.bdm.0 <- ncss.bdm[ncss.bdm$prep_code=="S",]
  summary(ncss.bdm.0$db_od)
  ## 0 BD values --- error!
  ncss.carb <- read.csv("/mnt/diskstation/data/Soil_points/INT/USDA_NCSS/NCSS_Carbon_and_Extractions.csv", stringsAsFactors = FALSE)
  ncss.organic <- read.csv("/mnt/diskstation/data/Soil_points/INT/USDA_NCSS/NCSS_Organic.csv", stringsAsFactors = FALSE)
  ncss.pH <- read.csv("/mnt/diskstation/data/Soil_points/INT/USDA_NCSS/NCSS_pH_and_Carbonates.csv", stringsAsFactors = FALSE)
  #str(ncss.pH)
  #summary(ncss.pH$ph_h2o)
  #summary(!is.na(ncss.pH$ph_h2o))
  ncss.PSDA <- read.csv("/mnt/diskstation/data/Soil_points/INT/USDA_NCSS/NCSS_PSDA_and_Rock_Fragments.csv", stringsAsFactors = FALSE)
  ncss.CEC <- read.csv("/mnt/diskstation/data/Soil_points/INT/USDA_NCSS/NCSS_CEC_and_Bases.csv")
  ncss.salt <- read.csv("/mnt/diskstation/data/Soil_points/INT/USDA_NCSS/NCSS_Salt.csv")
  ncss.horizons <- plyr::join_all(list(ncss.bdm.0, ncss.layer, ncss.carb, ncss.organic[,c("labsampnum", "result_source_key", "c_tot", "n_tot", "db_od", "oc")], ncss.pH, ncss.PSDA, ncss.CEC, ncss.salt), type = "full", by="labsampnum")
  #head(ncss.horizons)
  nrow(ncss.horizons)
  ncss.horizons$oc_d = signif(ncss.horizons$oc / 100 * ncss.horizons$db_od * 1000 * (100 - ifelse(is.na(ncss.horizons$wpg2), 0, ncss.horizons$wpg2))/100, 3)
  ncss.horizons$ca_ext = signif(ncss.horizons$ca_nh4 * 200, 4)
  ncss.horizons$mg_ext = signif(ncss.horizons$mg_nh4 * 121, 3)
  ncss.horizons$na_ext = signif(ncss.horizons$na_nh4 * 230, 3)
  ncss.horizons$k_ext = signif(ncss.horizons$k_nh4 * 391, 3)
  #summary(ncss.horizons$oc_d)
  ## Values <0!!
  chemsprops.NCSS = plyr::join(ncss.site[,site.names], ncss.horizons[,hor.names], by="site_key") 
  chemsprops.NCSS$site_obsdate = format(as.Date(chemsprops.NCSS$site_obsdate, format="%m/%d/%Y"), "%Y-%m-%d")
  chemsprops.NCSS$source_db = "USDA_NCSS"
  #dim(chemsprops.NCSS)
  chemsprops.NCSS$oc = chemsprops.NCSS$oc * 10
  chemsprops.NCSS$n_tot = chemsprops.NCSS$n_tot * 10
  #hist(log1p(chemsprops.NCSS$oc), breaks=45, col="gray")
  chemsprops.NCSS$confidence_degree = 1
  chemsprops.NCSS$project_url = "http://ncsslabdatamart.sc.egov.usda.gov/"
  chemsprops.NCSS$citation_url = "https://doi.org/10.2136/sssaj2016.11.0386n"
  chemsprops.NCSS = complete.vars(chemsprops.NCSS, sel=c("tex_psda","oc","clay_tot_psa","ecec","ph_h2o","ec_12pre","k_ext"))
  #rm(ncss.horizons)
}
dim(chemsprops.NCSS)
#> [1] 136011     36
#summary(!is.na(chemsprops.NCSS$oc))
## texture classes need to be cleaned-up
summary(as.factor(chemsprops.NCSS$tex_psda))
#>                               c               C              cl              CL 
#>            2391           10908              19           10158               5 
#>             cos             CoS             COS            cosl            CoSL 
#>            2424               6               4            4543               2 
#> Fine Sandy Loam              fs             fsl             FSL               l 
#>               1            2357           10701              16           17038 
#>               L            lcos            LCoS             lfs              ls 
#>               2            2933               3            1805            3166 
#>              LS            lvfs               s               S              sc 
#>               1             152            2958               1             601 
#>             scl             SCL              si             sic             SiC 
#>            5456               5             854            7123              11 
#>            sicl            SiCL             sil             SiL             SIL 
#>           13718              14           22230               4              28 
#>              sl              SL             vfs            vfsl            VFSL 
#>            5547              13              64            2678               3 
#>            NA's 
#>            6068
```

#### Rapid Carbon Assessment (RaCA)

- Soil Survey Staff. Rapid Carbon Assessment (RaCA) project. United States Department of Agriculture, Natural Resources Conservation Service. Available online. June 1, 2013 (FY2013 official release). Data download URL: https://www.nrcs.usda.gov/wps/portal/nrcs/detailfull/soils/research/?cid=nrcs142p2_054164
- **Note**: Locations of each site have been degraded due to confidentiality and only reflect the general position of each site.
- Wills, S. et al. (2013) ["Rapid carbon assessment (RaCA) methodology: Sampling and Initial Summary. United States Department of Agriculture."](https://www.nrcs.usda.gov/wps/PA_NRCSConsumption/download?cid=nrcs142p2_052841&ext=pdf) Natural Resources Conservation Service, National Soil Survey Center.



```r
if(!exists("chemsprops.RaCa")){
  raca.df <- read.csv("/mnt/diskstation/data/Soil_points/USA/RaCA/RaCa_general_location.csv", stringsAsFactors = FALSE)
  names(raca.df)[1] = "rcasiteid"
  raca.layer <- read.csv("/mnt/diskstation/data/Soil_points/USA/RaCA/RaCA_samples_JULY2016.csv", stringsAsFactors = FALSE)
  raca.layer$longitude_decimal_degrees = plyr::join(raca.layer["rcasiteid"], raca.df, match ="first")$Gen_long
  raca.layer$latitude_decimal_degrees = plyr::join(raca.layer["rcasiteid"], raca.df, match ="first")$Gen_lat
  raca.layer$site_obsdate = "2013"
  summary(raca.layer$Calc_SOC)
  #plot(raca.layer[!duplicated(raca.layer$rcasiteid),c("longitude_decimal_degrees", "latitude_decimal_degrees")])
  #summary(raca.layer$SOC_pred1)
  ## some strange groupings around small values
  raca.layer$oc_d = signif(raca.layer$Calc_SOC / 100 * raca.layer$Bulkdensity * 1000 * (100 - ifelse(is.na(raca.layer$fragvolc), 0, raca.layer$fragvolc))/100, 3)
  raca.layer$oc = raca.layer$Calc_SOC * 10
  #summary(raca.layer$oc_d)
  raca.h.lst <- c("rcasiteid", "lay_field_label1", "site_obsdate", "longitude_decimal_degrees", "latitude_decimal_degrees", "Lab.Sample.No", "layer_Number", "TOP", "BOT", "hzname", "texture", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", "oc", "oc_d", "c_tot_ncs", "n_tot_ncs", "ph_kcl", "ph_h2o", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "fragvolc", "Bulkdensity", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = raca.h.lst[which(!raca.h.lst %in% names(raca.layer))]
  if(length(x.na)>0){ for(i in x.na){ raca.layer[,i] = NA } }
  chemsprops.RaCA = raca.layer[,raca.h.lst]
  chemsprops.RaCA$source_db = "RaCA2016"
  chemsprops.RaCA$confidence_degree = 4
  chemsprops.RaCA$project_url = "https://www.nrcs.usda.gov/survey/raca/"
  chemsprops.RaCA$citation_url = "https://www.nrcs.usda.gov/Internet/FSE_DOCUMENTS/nrcs142p2_052841.pdf"
  chemsprops.RaCA = complete.vars(chemsprops.RaCA, sel = c("oc", "fragvolc"))
}
#> Joining by: rcasiteid
#> Joining by: rcasiteid
dim(chemsprops.RaCA)
#> [1] 53664    36
```

#### National Geochemical Database Soil

- Smith, D.B., Cannon, W.F., Woodruff, L.G., Solano, Federico, Kilburn, J.E., and Fey, D.L., (2013). [Geochemical and
mineralogical data for soils of the conterminous United States](http://pubs.usgs.gov/ds/801/). U.S. Geological Survey Data Series 801, 19 p., http://pubs.usgs.gov/ds/801/.
- Grossman, J. N. (2004). [The National Geochemical Survey-database and documentation](https://doi.org/10.3133/ofr20041001). U.S. Geological Survey Open-File Report 2004-1001. DOI:10.3133/ofr20041001.
- **Note**: NGS focuses on stream-sediment samples, but also contains many soil samples.


```r
if(!exists("chemsprops.USGS.NGS")){
  ngs.points <- read.csv("/mnt/diskstation/data/Soil_points/USA/geochemical/ds-801-csv/site.txt", sep=",")
  ## 4857 pnts
  ngs.layers <- lapply(c("top5cm.txt", "ahorizon.txt", "chorizon.txt"), function(i){read.csv(paste0("/mnt/diskstation/data/Soil_points/USA/geochemical/ds-801-csv/", i), sep=",")})
  ngs.layers = plyr::rbind.fill(ngs.layers)
  #dim(ngs.layers)
  # 14571  126
  #summary(ngs.layers$tot_carb_pct)
  #lattice::xyplot(c_org_pct ~ c_tot_pct, ngs.layers, scales=list(x = list(log = 2), y = list(log = 2)))
  #lattice::xyplot(c_org_pct ~ tot_clay_pct, ngs.layers, scales=list(y = list(log = 2)))
  ngs.layers$c_tot = ngs.layers$c_tot_pct * 10
  ngs.layers$oc = ngs.layers$c_org_pct * 10
  ngs.layers$hzn_top = sapply(ngs.layers$depth_cm, function(i){strsplit(i, "-")[[1]][1]})
  ngs.layers$hzn_bot = sapply(ngs.layers$depth_cm, function(i){strsplit(i, "-")[[1]][2]})
  #summary(ngs.layers$tot_clay_pct)
  #summary(ngs.layers$k_pct) ## very high numbers?
  ## question is if the geochemical element results are compatible with e.g. k_ext?
  t.ngs = c("lab_id", "site_id", "horizon", "hzn_top", "hzn_bot", "tot_clay_pct", "c_tot", "oc")
  ngs.m = plyr::join(ngs.points, ngs.layers[!is.na(ngs.layers$c_org_pct),t.ngs])
  ngs.m$site_obsdate = as.Date(ngs.m$colldate, format="%Y-%m-%d") 
  ngs.h.lst <- c("site_id", "quad", "site_obsdate", "longitude", "latitude", "lab_id", "layer_sequence", "hzn_top", "hzn_bot", "horizon", "tex_psda", "tot_clay_pct", "silt_tot_psa", "sand_tot_psa", "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "wpg2", "db_od", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = ngs.h.lst[which(!ngs.h.lst %in% names(ngs.m))]
  if(length(x.na)>0){ for(i in x.na){ ngs.m[,i] = NA } }
  chemsprops.USGS.NGS = ngs.m[,ngs.h.lst]
  chemsprops.USGS.NGS$source_db = "USGS.NGS"
  chemsprops.USGS.NGS$confidence_degree = 1
  chemsprops.USGS.NGS$project_url = "https://mrdata.usgs.gov/ds-801/"
  chemsprops.USGS.NGS$citation_url = "https://pubs.usgs.gov/ds/801/"
  chemsprops.USGS.NGS = complete.vars(chemsprops.USGS.NGS, sel = c("tot_clay_pct", "oc"), coords = c("longitude", "latitude"))
}
dim(chemsprops.USGS.NGS)
#> [1] 9446   36
```

#### Forest Inventory and Analysis Database (FIADB)

- Domke, G. M., Perry, C. H., Walters, B. F., Nave, L. E., Woodall, C. W., & Swanston, C. W. (2017). [Toward inventory‐based estimates of soil organic carbon in forests of the United States](https://doi.org/10.1002/eap.1516). Ecological Applications, 27(4), 1223-1235. https://doi.org/10.1002/eap.1516
- Forest Inventory and Analysis, (2014). The Forest Inventory and Analysis Database: Database description
and user guide version 6.0.1 for Phase 3. U.S. Department of Agriculture, Forest Service. 182 p.
[Online]. Available: https://www.fia.fs.fed.us/library/database-documentation/
- **Note**: samples are taken only from the top-soil either 0–10.16 cm or 10.16–20.32 cm.


```r
if(!exists("chemsprops.FIADB")){
  fia.loc <- vroom::vroom("/mnt/diskstation/data/Soil_points/USA/FIADB/ENTIRE/PLOT.csv")
  fia.loc$site_id = paste(fia.loc$STATECD, fia.loc$COUNTYCD, fia.loc$PLOT, sep="_")
  fia.lab <- read.csv("/mnt/diskstation/data/Soil_points/USA/FIADB/ENTIRE/SOILS_LAB.csv")
  fia.lab$site_id = paste(fia.lab$STATECD, fia.lab$COUNTYCD, fia.lab$PLOT, sep="_")
  ## 23,765 rows
  fia.des <- read.csv("/mnt/diskstation/data/Soil_points/USA/FIADB/ENTIRE/SOILS_SAMPLE_LOC.csv")
  fia.des$site_id = paste(fia.des$STATECD, fia.des$COUNTYCD, fia.des$PLOT, sep="_")
  #fia.lab$TXTRLYR1 = plyr::join(fia.lab[c("site_id","INVYR")], fia.des[c("site_id","TXTRLYR1","INVYR")], match ="first")$TXTRLYR1
  fia.lab$TXTRLYR2 = plyr::join(fia.lab[c("site_id","INVYR")], fia.des[c("site_id","TXTRLYR2","INVYR")], match ="first")$TXTRLYR2
  #summary(as.factor(fia.lab$TXTRLYR1))
  fia.lab$tex_psda = factor(fia.lab$TXTRLYR2, labels = c("Organic", "Loamy", "Clayey", "Sandy", "Coarse sand", "Not measured")) 
  #Code Description
  # 0 Organic.
  # 1 Loamy.
  # 2 Clayey.
  # 3 Sandy.
  # 4 Coarse sand.
  # 9 Not measured - make plot notes
  fia.lab$FORFLTHK = plyr::join(fia.lab[c("site_id","INVYR")], fia.des[c("site_id","FORFLTHK","INVYR")], match ="first")$FORFLTHK
  #summary(fia.lab$FORFLTHK)
  fia.lab$LTRLRTHK = plyr::join(fia.lab[c("site_id","INVYR")], fia.des[c("site_id","LTRLRTHK","INVYR")], match ="first")$LTRLRTHK
  fia.lab$tot_thk = rowSums(fia.lab[,c("FORFLTHK", "LTRLRTHK")], na.rm=TRUE)
  fia.lab$DPTHSBSL = plyr::join(fia.lab[c("site_id","INVYR")], fia.des[c("site_id","DPTHSBSL","INVYR")], match ="first")$DPTHSBSL
  #summary(fia.lab$DPTHSBSL)
  sel.fia = fia.loc$site_id %in% fia.lab$site_id
  #summary(sel.fia)
  # 15,109
  fia.loc = fia.loc[sel.fia, c("site_id", "LON", "LAT")]
  #summary(fia.lab$BULK_DENSITY)  ## some strange values for BD!
  #quantile(fia.lab$BULK_DENSITY, c(0.02, 0.98), na.rm=TRUE)
  #summary(fia.lab$C_ORG_PCT)
  #summary(as.factor(fia.lab$LAYER_TYPE))
  #lattice::xyplot(BULK_DENSITY ~ C_ORG_PCT, fia.lab, scales=list(x = list(log = 2)))
  #dim(fia.lab)
  # 14571  126
  fia.lab$c_tot = fia.lab$C_ORG_PCT * 10
  fia.lab$oc = fia.lab$C_TOTAL_PCT * 10
  fia.lab$n_tot = fia.lab$N_TOTAL_PCT * 10
  fia.lab$db_od = ifelse(fia.lab$BULK_DENSITY < 0.001 | fia.lab$BULK_DENSITY > 1.8, NA, fia.lab$BULK_DENSITY)
  #lattice::xyplot(db_od ~ C_ORG_PCT, fia.lab, par.settings = list(plot.symbol = list(col=scales::alpha("black", 0.6), fill=scales::alpha("red", 0.6), pch=21, cex=0.6)), scales = list(x=list(log=TRUE, equispaced.log=FALSE)), ylab="Bulk density", xlab="SOC wpct")
  #hist(fia.lab$db_od, breaks=45)
  ## A lot of very small BD measurements
  fia.lab$oc_d = signif(fia.lab$oc / 100 * fia.lab$db_od * 1000 * (100 - ifelse(is.na(fia.lab$COARSE_FRACTION_PCT), 0, fia.lab$COARSE_FRACTION_PCT))/100, 3)
  #hist(fia.lab$oc_d, breaks=45, col="grey")
  fia.lab$hzn_top = ifelse(fia.lab$LAYER_TYPE=="FF_TOTAL" | fia.lab$LAYER_TYPE=="L_ORG", 0, NA)
  fia.lab$hzn_bot = ifelse(fia.lab$LAYER_TYPE=="FF_TOTAL" | fia.lab$LAYER_TYPE=="L_ORG", fia.lab$tot_thk, NA)
  fia.lab$hzn_top = ifelse(is.na(fia.lab$hzn_top) & (fia.lab$LAYER_TYPE=="MIN_2" | fia.lab$LAYER_TYPE=="ORG_2"), 10.2 + fia.lab$tot_thk, fia.lab$hzn_top)
  fia.lab$hzn_bot = ifelse(is.na(fia.lab$hzn_bot) & (fia.lab$LAYER_TYPE=="MIN_2" | fia.lab$LAYER_TYPE=="ORG_2"), 20.3 + fia.lab$tot_thk, fia.lab$hzn_bot)
  fia.lab$hzn_top = ifelse(is.na(fia.lab$hzn_top) & (fia.lab$LAYER_TYPE=="MIN_1" | fia.lab$LAYER_TYPE=="ORG_1"), 0 + fia.lab$tot_thk, fia.lab$hzn_top)
  fia.lab$hzn_bot = ifelse(is.na(fia.lab$hzn_bot) & (fia.lab$LAYER_TYPE=="MIN_1" | fia.lab$LAYER_TYPE=="ORG_1"), 10.2 + fia.lab$tot_thk, fia.lab$hzn_bot) 
  #summary(fia.lab$EXCHNG_K) ## Negative values!
  fia.m = plyr::join(fia.lab, fia.loc)
  #fia.m = fia.m[!duplicated(as.factor(paste(fia.m$site_id, fia.m$INVYR, fia.m$LAYER_TYPE, sep="_"))),]
  fia.m$site_obsdate = as.Date(fia.m$SAMPLE_DATE, format="%Y-%m-%d")
  sel.d.fia = fia.m$site_obsdate < as.Date("1980-01-01", format="%Y-%m-%d")
  fia.m$site_obsdate[which(sel.d.fia)] = NA
  #hist(fia.m$site_obsdate, breaks=25)
  fia.h.lst <- c("site_id", "usiteid", "site_obsdate", "LON", "LAT", "SAMPLE_ID", "layer_sequence", "hzn_top", "hzn_bot", "LAYER_TYPE", "tex_psda", "tot_clay_pct", "silt_tot_psa", "sand_tot_psa", "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "PH_H2O", "PH_CACL2", "cec_sum", "cec_nh4", "ECEC", "COARSE_FRACTION_PCT", "db_od", "EXCHNG_CA", "EXCHNG_MG", "EXCHNG_NA", "EXCHNG_K", "ec_satp", "ec_12pre")
  x.na = fia.h.lst[which(!fia.h.lst %in% names(fia.m))]
  if(length(x.na)>0){ for(i in x.na){ fia.m[,i] = NA } }
  chemsprops.FIADB = fia.m[,fia.h.lst]
  chemsprops.FIADB$source_db = "FIADB"
  chemsprops.FIADB$confidence_degree = 2
  chemsprops.FIADB$project_url = "http://www.fia.fs.fed.us/"
  chemsprops.FIADB$citation_url = "https://www.fia.fs.fed.us/library/database-documentation/"
  chemsprops.FIADB = complete.vars(chemsprops.FIADB, sel = c("PH_H2O", "oc", "EXCHNG_K"), coords = c("LON", "LAT"))
  #str(unique(paste(chemsprops.FIADB$LON, chemsprops.FIADB$LAT, sep="_")))
}
dim(chemsprops.FIADB)
#> [1] 23208    36
#write.csv(chemsprops.FIADB, "/mnt/diskstation/data/Soil_points/USA/FIADB/fiadb_soil.pnts.csv")
```


#### Africa soil profiles database

- Leenaars, J. G., Van Oostrum, A. J. M., & Ruiperez Gonzalez, M. (2014). [Africa soil profiles database version 1.2. A compilation of georeferenced and standardized legacy soil profile data for Sub-Saharan Africa (with dataset)](https://www.isric.org/projects/africa-soil-profiles-database-afsp). Wageningen: ISRIC Report 2014/01; 2014. Data download URL: https://data.isric.org/


```r
if(!exists("chemsprops.AfSPDB")){
  library(foreign)
  afspdb.profiles <- read.dbf("/mnt/diskstation/data/Soil_points/AF/AfSIS_SPDB/AfSP012Qry_Profiles.dbf", as.is=TRUE)
  afspdb.layers <- read.dbf("/mnt/diskstation/data/Soil_points/AF/AfSIS_SPDB/AfSP012Qry_Layers.dbf", as.is=TRUE)
  afspdb.s.lst <- c("ProfileID", "FldMnl_ID", "T_Year", "X_LonDD", "Y_LatDD")
  #summary(afspdb.layers$BlkDens)
  ## add missing columns
  for(j in 1:ncol(afspdb.layers)){ 
    if(is.numeric(afspdb.layers[,j])) { 
      afspdb.layers[,j] <- ifelse(afspdb.layers[,j] < 0, NA, afspdb.layers[,j]) 
    }
  }
  afspdb.layers$ca_ext = afspdb.layers$ExCa * 200
  afspdb.layers$mg_ext = afspdb.layers$ExMg * 121
  afspdb.layers$na_ext = afspdb.layers$ExNa * 230
  afspdb.layers$k_ext = afspdb.layers$ExK * 391
  #summary(afspdb.layers$k_ext)
  afspdb.m = plyr::join(afspdb.profiles[,afspdb.s.lst], afspdb.layers)
  afspdb.m$oc_d = signif(afspdb.m$OrgC * afspdb.m$BlkDens * (100 - ifelse(is.na(afspdb.m$CfPc), 0, afspdb.m$CfPc))/100, 3)
  #summary(afspdb.m$T_Year)
  afspdb.m$T_Year = ifelse(afspdb.m$T_Year < 0, NA, afspdb.m$T_Year)
  afspdb.h.lst <- c("ProfileID", "FldMnl_ID", "T_Year", "X_LonDD", "Y_LatDD", "LayerID", "LayerNr", "UpDpth", "LowDpth", "HorDes", "LabTxtr", "Clay", "Silt", "Sand", "OrgC", "oc_d", "TotC", "TotalN", "PHKCl", "PHH2O", "PHCaCl2", "CecSoil", "cec_nh4", "Ecec", "CfPc" , "BlkDens", "ca_ext", "mg_ext", "na_ext", "k_ext", "EC", "ec_12pre")
  x.na = afspdb.h.lst[which(!afspdb.h.lst %in% names(afspdb.m))]
  if(length(x.na)>0){ for(i in x.na){ afspdb.m[,i] = NA } }
  chemsprops.AfSPDB = afspdb.m[,afspdb.h.lst]
  chemsprops.AfSPDB$source_db = "AfSPDB"
  chemsprops.AfSPDB$confidence_degree = 5
  chemsprops.AfSPDB$project_url = "https://www.isric.org/projects/africa-soil-profiles-database-afsp"
  chemsprops.AfSPDB$citation_url = "https://www.isric.org/sites/default/files/isric_report_2014_01.pdf"
  chemsprops.AfSPDB = complete.vars(chemsprops.AfSPDB, sel = c("LabTxtr","OrgC","Clay","Ecec","PHH2O","EC","k_ext"), coords = c("X_LonDD", "Y_LatDD"))
}
dim(chemsprops.AfSPDB)
#> [1] 60306    36
```

#### Africa Soil Information Service (AfSIS) Soil Chemistry

- Towett, E. K., Shepherd, K. D., Tondoh, J. E., Winowiecki, L. A., Lulseged, T., Nyambura, M., ... & Cadisch, G. (2015). Total elemental composition of soils in Sub-Saharan Africa and relationship with soil forming factors. Geoderma Regional, 5, 157-168. https://doi.org/10.1016/j.geodrs.2015.06.002
- [AfSIS Soil Chemistry](https://github.com/qedsoftware/afsis-soil-chem-tutorial) produced by World Agroforestry Centre (ICRAF), Quantitative Engineering Design (QED), Center for International Earth Science Information Network (CIESIN), The International Center for Tropical Agriculture (CIAT), Crop Nutrition Laboratory Services (CROPNUTS) and Rothamsted Research (RRES). Data download URL: https://registry.opendata.aws/afsis/


```r
if(!exists("chemsprops.AfSIS1")){
  afsis1.xy = read.csv("/mnt/diskstation/data/Soil_points/AF/AfSIS_SSL/2009-2013/Georeferences/georeferences.csv")
  afsis1.xy$Sampling.date = 2011
  afsis1.lst = list.files("/mnt/diskstation/data/Soil_points/AF/AfSIS_SSL/2009-2013/Wet_Chemistry", pattern=glob2rx("*.csv$"), full.names = TRUE, recursive = TRUE)
  afsis1.hor = plyr::rbind.fill(lapply(afsis1.lst, read.csv))
  tansis.xy = read.csv("/mnt/diskstation/data/Soil_points/AF/AfSIS_SSL/tansis/Georeferences/georeferences.csv")
  #summary(tansis.xy$Sampling.date)
  tansis.xy$Sampling.date = 2018
  tansis.lst = list.files("/mnt/diskstation/data/Soil_points/AF/AfSIS_SSL/tansis/Wet_Chemistry", pattern=glob2rx("*.csv$"), full.names = TRUE, recursive = TRUE)
  tansis.hor = plyr::rbind.fill(lapply(tansis.lst, read.csv))
  afsis1t.df = plyr::rbind.fill(list(plyr::join(afsis1.hor, afsis1.xy, by="SSN"), plyr::join(tansis.hor, tansis.xy, by="SSN")))
  afsis1t.df$UpDpth = ifelse(afsis1t.df$Depth=="sub", 20, 0)
  afsis1t.df$LowDpth = ifelse(afsis1t.df$Depth=="sub", 50, 20)
  afsis1t.df$LayerNr = ifelse(afsis1t.df$Depth=="sub", 2, 1)
  #summary(afsis1t.df$C...Org)
  afsis1t.df$oc = rowMeans(afsis1t.df[,c("C...Org", "X.C")], na.rm=TRUE) * 10
  afsis1t.df$c_tot = afsis1t.df$Total.carbon
  afsis1t.df$n_tot = rowMeans(afsis1t.df[,c("Total.nitrogen", "X.N")], na.rm=TRUE) * 10
  afsis1t.df$ph_h2o = rowMeans(afsis1t.df[,c("PH", "pH")], na.rm=TRUE)
  ## multiple texture fractons - which one is the total clay, sand, silt?
  ## Clay content for water dispersed particles-recorded after 4 minutes of ultrasonication
  #summary(afsis1t.df$Psa.w4clay)
  #plot(afsis1t.df[,c("Longitude", "Latitude")])
  afsis1.h.lst <- c("SSN", "Site", "Sampling.date", "Longitude", "Latitude", "Soil.material", "LayerNr", "UpDpth", "LowDpth", "HorDes", "LabTxtr", "Psa.w4clay", "Psa.w4silt", "Psa.w4sand", "oc", "oc_d", "c_tot", "n_tot", "PHKCl", "ph_h2o", "PHCaCl2", "CecSoil", "cec_nh4", "Ecec", "CfPc" , "BlkDens", "ca_ext", "M3.Mg", "M3.Na", "M3.K", "EC", "ec_12pre")
  x.na = afspdb.h.lst[which(!afsis1.h.lst %in% names(afsis1t.df))]
  if(length(x.na)>0){ for(i in x.na){ afsis1t.df[,i] = NA } }
  chemsprops.AfSIS1 = afsis1t.df[,afsis1.h.lst]
  chemsprops.AfSIS1$source_db = "AfSIS1"
  chemsprops.AfSIS1$confidence_degree = 2
  chemsprops.AfSIS1$project_url = "https://registry.opendata.aws/afsis/"
  chemsprops.AfSIS1$citation_url = "https://doi.org/10.1016/j.geodrs.2015.06.002"
  chemsprops.AfSIS1 = complete.vars(chemsprops.AfSIS1, sel = c("Psa.w4clay","oc","ph_h2o","M3.K"), coords = c("Longitude", "Latitude"))
}
dim(chemsprops.AfSIS1)
#> [1] 4162   36
```

#### Fine Root Ecology Database (FRED)

- Iversen CM, McCormack ML, Baer JK, Powell AS, Chen W, Collins C, Fan Y, Fanin N, Freschet GT, Guo D, Hogan JA, Kou L, Laughlin DC, Lavely E, Liese R, Lin D, Meier IC, Montagnoli A, Roumet C, See CR, Soper F, Terzaghi M, Valverde-Barrantes OJ, Wang C, Wright SJ, Wurzburger N, Zadworny M. (2021). [Fine-Root Ecology Database (FRED): A Global Collection of Root Trait Data with Coincident Site, Vegetation, Edaphic, and Climatic Data, Version 3](https://roots.ornl.gov/). Oak Ridge National Laboratory, TES SFA, U.S. Department of Energy, Oak Ridge, Tennessee, U.S.A. Access on-line at: https://doi.org/10.25581/ornlsfa.014/1459186.


```r
if(!exists("chemsprops.FRED")){
  Sys.setenv("VROOM_CONNECTION_SIZE" = 131072 * 2)
  fred = vroom::vroom("/mnt/diskstation/data/Soil_points/INT/FRED/FRED3_Entire_Database_2021.csv", skip = 10, col_names=FALSE)
  ## 57,190 x 1,164
  #nm.fred = read.csv("/mnt/diskstation/data/Soil_points/INT/FRED/FRED3_Column_Definitions_20210423-091040.csv", header=TRUE)
  nm.fred0 = read.csv("/mnt/diskstation/data/Soil_points/INT/FRED/FRED3_Entire_Database_2021.csv", nrows=2)
  names(fred) = make.names(t(nm.fred0)[,1])
  ## 1164 columns!
  fred.h.lst = c("Notes_Row.ID", "Data.source_DOI", "site_obsdate", "longitude_decimal_degrees", "latitude_decimal_degrees", "labsampnum", "layer_sequence", "hzn_top", "hzn_bot", "Soil.horizon", "Soil.texture", "Soil.texture_Fraction.clay", "Soil.texture_Fraction.silt", "Soil.texture_Fraction.sand", "Soil.organic.C.content", "oc_d", "c_tot", "Soil.N.content", "ph_kcl", "Soil.pH_Water", "Soil.pH_Salt", "Soil.cation.exchange.capacity..CEC.", "cec_nh4", "Soil.effective.cation.exchange.capacity..ECEC.", "wpg2", "Soil.bulk.density", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre", "source_db", "confidence_degree")
  fred$site_obsdate = as.integer(rowMeans(fred[,c("Sample.collection_Year.ending.collection", "Sample.collection_Year.beginning.collection")], na.rm=TRUE))
  #summary(fred$site_obsdate)
  fred$longitude_decimal_degrees = ifelse(is.na(fred$Longitude), fred$Longitude_Estimated, fred$Longitude)
  fred$latitude_decimal_degrees = ifelse(is.na(fred$Latitude), fred$Latitude_Estimated, fred$Latitude)
  #names(fred)[grep("Notes_Row", names(fred))]
  #summary(fred[,grep("clay", names(fred))])
  #summary(fred[,grep("cation.exchange", names(fred))])
  #summary(fred[,grep("organic.C", names(fred))])
  #summary(fred$Soil.organic.C.content)
  #summary(fred$Soil.bulk.density)
  #summary(as.factor(fred$Soil.horizon))
  fred$hzn_bot = ifelse(is.na(fred$Soil.depth_Lower.sampling.depth), fred$Soil.depth - 5, fred$Soil.depth_Lower.sampling.depth)
  fred$hzn_top = ifelse(is.na(fred$Soil.depth_Upper.sampling.depth), fred$Soil.depth + 5, fred$Soil.depth_Upper.sampling.depth)
  fred$oc_d = signif(fred$Soil.organic.C.content / 1000 * fred$Soil.bulk.density * 1000, 3)
  #summary(fred$oc_d)  
  x.na = fred.h.lst[which(!fred.h.lst %in% names(fred))]
  if(length(x.na)>0){ for(i in x.na){ fred[,i] = NA } }
  chemsprops.FRED = fred[,fred.h.lst]
  #plot(chemsprops.FRED[,4:5])
  chemsprops.FRED$source_db = "FRED"
  chemsprops.FRED$confidence_degree = 5
  chemsprops.FRED$project_url = "https://roots.ornl.gov/"
  chemsprops.FRED$citation_url = "https://doi.org/10.25581/ornlsfa.014/1459186"
  chemsprops.FRED = complete.vars(chemsprops.FRED, sel = c("Soil.organic.C.content", "Soil.texture_Fraction.clay", "Soil.pH_Water"))
  ## many duplicates
}
dim(chemsprops.FRED)
#> [1] 858  36
```

#### Global root traits (GRooT) database (compilation)

- Guerrero‐Ramírez, N. R., Mommer, L., Freschet, G. T., Iversen, C. M., McCormack, M. L., Kattge, J., ... & Weigelt, A. (2021). [Global root traits (GRooT) database](https://dx.doi.org/10.1111/geb.13179). Global ecology and biogeography, 30(1), 25-37. https://dx.doi.org/10.1111/geb.13179


```r
if(!exists("chemsprops.GROOT")){
  #Sys.setenv("VROOM_CONNECTION_SIZE" = 131072 * 2)
  GROOT = vroom::vroom("/mnt/diskstation/data/Soil_points/INT/GRooT/GRooTFullVersion.csv")
  ## 114,222 x 73
  c("locationID", "GRooTID", "originalID", "source", "year", "decimalLatitude", "decimalLongitud", "soilpH", "soilTexture", "soilCarbon", "soilNitrogen", "soilPhosphorus", "soilCarbonToNitrogen", "soilBaseCationSaturation", "soilCationExchangeCapacity", "soilOrganicMatter", "soilWaterGravimetric", "soilWaterVolumetric")
  #summary(GROOT$soilCarbon)
  #summary(!is.na(GROOT$soilCarbon))
  #summary(GROOT$soilOrganicMatter)
  #summary(GROOT$soilNitrogen)
  #summary(GROOT$soilpH)
  #summary(as.factor(GROOT$soilTexture))
  #lattice::xyplot(soilCarbon ~ soilpH, GROOT, par.settings = list(plot.symbol = list(col=scales::alpha("black", 0.6), fill=scales::alpha("red", 0.6), pch=21, cex=0.6)), scales = list(y=list(log=TRUE, equispaced.log=FALSE)), ylab="SOC", xlab="pH")
  GROOT$site_obsdate = as.Date(paste0(GROOT$year, "-01-01"), format="%Y-%m-%d")
  GROOT$hzn_top = 0
  GROOT$hzn_bot = 30
  GROOT.h.lst = c("locationID", "originalID", "site_obsdate", "decimalLongitud", "decimalLatitude", "GRooTID", "layer_sequence", "hzn_top", "hzn_bot", "hzn_desgn", "soilTexture", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", "soilCarbon", "oc_d", "c_tot", "soilNitrogen", "ph_kcl", "soilpH", "ph_cacl2", "soilCationExchangeCapacity", "cec_nh4", "ecec", "wpg2", "db_od", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = GROOT.h.lst[which(!GROOT.h.lst %in% names(GROOT))]
  if(length(x.na)>0){ for(i in x.na){ GROOT[,i] = NA } }
  chemsprops.GROOT = GROOT[,GROOT.h.lst]
  chemsprops.GROOT$source_db = "GROOT"
  chemsprops.GROOT$confidence_degree = 8
  chemsprops.GROOT$project_url = "https://groot-database.github.io/GRooT/"
  chemsprops.GROOT$citation_url = "https://dx.doi.org/10.1111/geb.13179"
  chemsprops.GROOT = complete.vars(chemsprops.GROOT, sel = c("soilCarbon", "soilpH"), coords = c("decimalLongitud", "decimalLatitude"))
}
dim(chemsprops.GROOT)
#> [1] 718  36
```


#### Global Soil Respiration DB

- Bond-Lamberty, B. and Thomson, A. (2010). A global database of soil respiration data, Biogeosciences, 7, 1915–1926, https://doi.org/10.5194/bg-7-1915-2010


```r
if(!exists("chemsprops.SRDB")){
  srdb = read.csv("/mnt/diskstation/data/Soil_points/INT/SRDB/srdb-data.csv")
  ## 10366 x 85
  srdb.h.lst = c("Site_ID", "Notes", "Study_midyear", "Longitude", "Latitude", "labsampnum", "layer_sequence", "hzn_top", "hzn_bot", "hzn_desgn", "tex_psd", "Soil_clay", "Soil_silt", "Soil_sand", "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "wpg2", "Soil_BD", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre", "source_db", "confidence_degree")
  #summary(srdb$Study_midyear)
  srdb$hzn_bot = ifelse(is.na(srdb$C_soildepth), 100, srdb$C_soildepth)
  srdb$hzn_top = 0
  #summary(srdb$Soil_clay)
  #summary(srdb$C_soilmineral)
  srdb$oc_d = signif(srdb$C_soilmineral / 1000 / (srdb$hzn_bot/100), 3)
  #summary(srdb$oc_d) 
  #summary(srdb$Soil_BD)
  srdb$oc = srdb$oc_d / srdb$Soil_BD
  #summary(srdb$oc)
  x.na = srdb.h.lst[which(!srdb.h.lst %in% names(srdb))]
  if(length(x.na)>0){ for(i in x.na){ srdb[,i] = NA } }
  chemsprops.SRDB = srdb[,srdb.h.lst]
  #plot(chemsprops.SRDB[,4:5])
  chemsprops.SRDB$source_db = "SRDB"
  chemsprops.SRDB$confidence_degree = 5
  chemsprops.SRDB$project_url = "https://github.com/bpbond/srdb/"
  chemsprops.SRDB$citation_url = "https://doi.org/10.5194/bg-7-1915-2010"
  chemsprops.SRDB = complete.vars(chemsprops.SRDB, sel = c("oc", "Soil_clay", "Soil_BD"), coords = c("Longitude", "Latitude"))
}
dim(chemsprops.SRDB)
#> [1] 1596   36
```

#### SOils DAta Harmonization database (SoDaH)

- Wieder, W. R., Pierson, D., Earl, S., Lajtha, K., Baer, S., Ballantyne, F., ... & Weintraub, S. (2020). [SoDaH: the SOils DAta Harmonization database, an open-source synthesis of soil data from research networks, version 1.0](https://doi.org/10.5194/essd-2020-195). Earth System Science Data Discussions, 1-19. https://doi.org/10.5194/essd-2020-195. Data download URL: https://doi.org/10.6073/pasta/9733f6b6d2ffd12bf126dc36a763e0b4


```r
if(!exists("chemsprops.SoDaH")){
  sodah.hor = read.csv("/mnt/diskstation/data/Soil_points/INT/SoDaH/521_soils_data_harmonization_6e8416fa0c9a2c2872f21ba208e6a919.csv")
  #head(sodah.hor)
  #summary(sodah.hor$coarse_frac)
  #summary(sodah.hor$lyr_soc)
  #summary(sodah.hor$lyr_som_WalkleyBlack/1.724)
  #summary(as.factor(sodah.hor$observation_date))
  sodah.hor$site_obsdate = as.integer(substr(sodah.hor$observation_date, 1, 4))
  sodah.hor$oc = ifelse(is.na(sodah.hor$lyr_soc), sodah.hor$lyr_som_WalkleyBlack/1.724, sodah.hor$lyr_soc) * 10
  sodah.hor$n_tot = sodah.hor$lyr_n_tot * 10
  sodah.hor$oc_d = signif(sodah.hor$oc / 1000 * sodah.hor$bd_samp * 1000 * (100 - ifelse(is.na(sodah.hor$coarse_frac), 0, sodah.hor$coarse_frac))/100, 3)
  sodah.hor$site_key = paste(sodah.hor$network, sodah.hor$location_name, sep="_")
  sodah.hor$labsampnum = make.unique(paste(sodah.hor$network, sodah.hor$location_name, sodah.hor$L1, sep="_"))
  #summary(sodah.hor$oc_d)
  sodah.h.lst = c("site_key", "data_file", "observation_date", "long", "lat", "labsampnum", "layer_sequence", "layer_top", "layer_bot", "hzn", "profile_texture_class", "clay", "silt", "sand", "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "ph_cacl", "cec_sum", "cec_nh4", "ecec", "coarse_frac", "bd_samp", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre", "source_db", "confidence_degree")
  x.na = sodah.h.lst[which(!sodah.h.lst %in% names(sodah.hor))]
  if(length(x.na)>0){ for(i in x.na){ sodah.hor[,i] = NA } }
  chemsprops.SoDaH = sodah.hor[,sodah.h.lst]
  #plot(chemsprops.SoDaH[,4:5])
  chemsprops.SoDaH$source_db = "SoDaH"
  chemsprops.SoDaH$confidence_degree = 3
  chemsprops.SoDaH$project_url = "https://lter.github.io/som-website"
  chemsprops.SoDaH$citation_url = "https://doi.org/10.5194/essd-2020-195"
  chemsprops.SoDaH = complete.vars(chemsprops.SoDaH, sel = c("oc", "clay", "ph_h2o"), coords = c("long", "lat"))
}
dim(chemsprops.SoDaH)
#> [1] 20383    36
```


#### ISRIC WISE harmonized soil profile data

- Batjes, N.H. (2019). [Harmonized soil profile data for applications at global and continental scales: updates to the WISE database](http://dx.doi.org/10.1111/j.1475-2743.2009.00202.x). Soil Use and Management 5:124–127. Data download URL: https://files.isric.org/public/wise/WD-WISE.zip


```r
if(!exists("chemsprops.WISE")){
  wise.site <- read.table("/mnt/diskstation/data/Soil_points/INT/ISRIC_WISE/WISE3_SITE.csv", sep=",", header=TRUE, stringsAsFactors = FALSE, fill=TRUE)
  wise.s.lst <- c("WISE3_id", "PITREF", "DATEYR", "LONDD", "LATDD")
  wise.site$LONDD = as.numeric(wise.site$LONDD)
  wise.site$LATDD = as.numeric(wise.site$LATDD)
  wise.layer <- read.table("/mnt/diskstation/data/Soil_points/INT/ISRIC_WISE/WISE3_HORIZON.csv", sep=",", header=TRUE, stringsAsFactors = FALSE, fill=TRUE)
  wise.layer$ca_ext = signif(wise.layer$EXCA * 200, 4)
  wise.layer$mg_ext = signif(wise.layer$EXMG * 121, 3)
  wise.layer$na_ext = signif(wise.layer$EXNA * 230, 3)
  wise.layer$k_ext = signif(wise.layer$EXK * 391, 3)
  wise.layer$oc_d = signif(wise.layer$ORGC / 1000 * wise.layer$BULKDENS * 1000 * (100 - ifelse(is.na(wise.layer$GRAVEL), 0, wise.layer$GRAVEL))/100, 3)
  wise.h.lst <- c("WISE3_ID", "labsampnum", "HONU", "TOPDEP", "BOTDEP", "DESIG", "tex_psda", "CLAY", "SILT", "SAND", "ORGC", "oc_d", "c_tot", "TOTN", "PHKCL", "PHH2O", "PHCACL2", "CECSOIL", "cec_nh4", "ecec", "GRAVEL" , "BULKDENS", "ca_ext", "mg_ext", "na_ext", "k_ext", "ECE", "ec_12pre")
  x.na = wise.h.lst[which(!wise.h.lst %in% names(wise.layer))]
  if(length(x.na)>0){ for(i in x.na){ wise.layer[,i] = NA } }
  chemsprops.WISE = merge(wise.site[,wise.s.lst], wise.layer[,wise.h.lst], by.x="WISE3_id", by.y="WISE3_ID")
  chemsprops.WISE$source_db = "ISRIC_WISE"
  chemsprops.WISE$confidence_degree = 4
  chemsprops.WISE$project_url = "https://isric.org"
  chemsprops.WISE$citation_url = "http://dx.doi.org/10.1111/j.1475-2743.2009.00202.x"
  chemsprops.WISE = complete.vars(chemsprops.WISE, sel = c("ORGC","CLAY","PHH2O","CECSOIL","k_ext"), coords = c("LONDD", "LATDD"))
}
dim(chemsprops.WISE)
#> [1] 23278    36
```

#### GEMAS

- Reimann, C., Fabian, K., Birke, M., Filzmoser, P., Demetriades, A., Négrel, P., ... & Anderson, M. (2018). [GEMAS: Establishing geochemical background and threshold for 53 chemical elements in European agricultural soil](https://doi.org/10.1016/j.apgeochem.2017.01.021). Applied Geochemistry, 88, 302-318. Data download URL: http://gemas.geolba.ac.at/ 


```r
if(!exists("chemsprops.GEMAS")){
  gemas.samples <- read.csv("/mnt/diskstation/data/Soil_points/EU/GEMAS/GEMAS.csv", stringsAsFactors = FALSE)
  ## GEMAS, agricultural soil, 0-20 cm, air dried, <2 mm, aqua regia Data from ACME, total C, TOC, CEC, ph_CaCl2
  gemas.samples$hzn_top = 0
  gemas.samples$hzn_bot = 20
  gemas.samples$oc = gemas.samples$TOC * 10
  #summary(gemas.samples$oc)
  gemas.samples$c_tot = gemas.samples$C_tot * 10
  gemas.samples$site_obsdate = 2009
  gemas.h.lst <- c("ID", "COUNRTY", "site_obsdate", "XCOO", "YCOO", "labsampnum", "layer_sequence", "hzn_top", "hzn_bot", "TYPE", "tex_psda", "clay", "silt", "sand_tot_psa", "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "pH_CaCl2", "CEC", "cec_nh4", "ecec", "wpg2", "db_od", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = gemas.h.lst[which(!gemas.h.lst %in% names(gemas.samples))]
  if(length(x.na)>0){ for(i in x.na){ gemas.samples[,i] = NA } }
  chemsprops.GEMAS <- gemas.samples[,gemas.h.lst]
  chemsprops.GEMAS$source_db = "GEMAS_2009"
  chemsprops.GEMAS$confidence_degree = 2
  chemsprops.GEMAS$project_url = "http://gemas.geolba.ac.at/"
  chemsprops.GEMAS$citation_url = "https://doi.org/10.1016/j.apgeochem.2017.01.021"
  chemsprops.GEMAS = complete.vars(chemsprops.GEMAS, sel = c("oc","clay","pH_CaCl2"), coords = c("XCOO", "YCOO"))
}
dim(chemsprops.GEMAS)
#> [1] 4131   36
```

#### LUCAS soil

- Orgiazzi, A., Ballabio, C., Panagos, P., Jones, A., & Fernández‐Ugalde, O. (2018). [LUCAS Soil, the largest expandable soil dataset for Europe: a review](https://doi.org/10.1111/ejss.12499). European Journal of Soil Science, 69(1), 140-153. Data download URL: https://esdac.jrc.ec.europa.eu/content/lucas-2009-topsoil-data


```r
if(!exists("chemsprops.LUCAS")){
  lucas.samples <- openxlsx::read.xlsx("/mnt/diskstation/data/Soil_points/EU/LUCAS/LUCAS_TOPSOIL_v1.xlsx", sheet = 1)
  lucas.samples$site_obsdate <- "2009"
  #summary(lucas.samples$N)
  lucas.ro <- openxlsx::read.xlsx("/mnt/diskstation/data/Soil_points/EU/LUCAS/Romania.xlsx", sheet = 1)
  lucas.ro$site_obsdate <- "2012"
  names(lucas.samples)[which(!names(lucas.samples) %in% names(lucas.ro))]
  lucas.ro = plyr::rename(lucas.ro, replace=c("Soil.ID"="sample_ID", "GPS_X_LONG"="GPS_LONG", "GPS_Y_LAT"="GPS_LAT", "pHinH2O"="pH_in_H2O", "pHinCaCl2"="pH_in_CaCl"))
  lucas.bu <- openxlsx::read.xlsx("/mnt/diskstation/data/Soil_points/EU/LUCAS/Bulgaria.xlsx", sheet = 1)
  lucas.bu$site_obsdate <- "2012"
  names(lucas.samples)[which(!names(lucas.samples) %in% names(lucas.bu))]
  #lucas.ch <- openxlsx::read.xlsx("/mnt/diskstation/data/Soil_points/EU/LUCAS/LUCAS_2015_Topsoil_data_of_Switzerland-with-coordinates.xlsx_.xlsx", sheet = 1, startRow = 2)
  #lucas.ch = plyr::rename(lucas.ch, replace=c("Soil_ID"="sample_ID", "GPS_.LAT"="GPS_LAT", "pH.in.H2O"="pH_in_H2O", "pH.in.CaCl2"="pH_in_CaCl", "Calcium.carbonate/.g.kg–1"="CaCO3", "Silt/.g.kg–1"="silt", "Sand/.g.kg–1"="sand", "Clay/.g.kg–1"="clay", "Organic.carbon/.g.kg–1"="OC"))
  ## Double readings?
  lucas.t = plyr::rbind.fill(list(lucas.samples, lucas.ro, lucas.bu))
  lucas.h.lst <- c("POINT_ID", "usiteid", "site_obsdate", "GPS_LONG", "GPS_LAT", "sample_ID", "layer_sequence", "hzn_top", "hzn_bot", "hzn_desgn", "tex_psda", "clay", "silt", "sand", "OC", "oc_d", "c_tot", "N", "ph_kcl", "pH_in_H2O", "pH_in_CaCl", "CEC", "cec_nh4", "ecec", "coarse", "db_od", "ca_ext", "mg_ext", "na_ext", "K", "ec_satp", "ec_12pre")
  x.na = lucas.h.lst[which(!lucas.h.lst %in% names(lucas.t))]
  if(length(x.na)>0){ for(i in x.na){ lucas.t[,i] = NA } }
  chemsprops.LUCAS <- lucas.t[,lucas.h.lst]
  chemsprops.LUCAS$source_db = "LUCAS_2009"
  chemsprops.LUCAS$hzn_top <- 0
  chemsprops.LUCAS$hzn_bot <- 20
  chemsprops.LUCAS$confidence_degree = 2
  chemsprops.LUCAS$project_url = "https://esdac.jrc.ec.europa.eu/"
  chemsprops.LUCAS$citation_url = "https://doi.org/10.1111/ejss.12499"
  chemsprops.LUCAS = complete.vars(chemsprops.LUCAS, sel = c("OC","clay","pH_in_H2O"), coords = c("GPS_LONG", "GPS_LAT"))
}
dim(chemsprops.LUCAS)
#> [1] 21272    36
```


```r
if(!exists("chemsprops.LUCAS2")){
  #lucas2015.samples <- openxlsx::read.xlsx("/mnt/diskstation/data/Soil_points/EU/LUCAS/LUCAS_Topsoil_2015_20200323.xlsx", sheet = 1)
  lucas2015.xy = readOGR("/mnt/diskstation/data/Soil_points/EU/LUCAS/LUCAS_Topsoil_2015_20200323.shp")
  #head(as.data.frame(lucas2015.xy))
  lucas2015.xy = as.data.frame(lucas2015.xy)
  ## https://www.aqion.de/site/130
  ## 1 mS/m = 100 mS/cm | 1 dS/m = 1 mS/cm = 1 mS/m / 100
  lucas2015.xy$ec_satp = lucas2015.xy$EC / 100
  lucas2015.h.lst <- c("Point_ID", "LC0_Desc", "site_obsdate", "coords.x1", "coords.x2", "sample_ID", "layer_sequence", "hzn_top", "hzn_bot", "hzn_desgn", "tex_psda", "Clay", "Silt", "Sand", "OC", "oc_d", "c_tot", "N", "ph_kcl", "pH_H20", "pH_CaCl2", "CEC", "cec_nh4", "ecec", "coarse", "db_od", "ca_ext", "mg_ext", "na_ext", "K", "ec_satp", "ec_12pre")
  x.na = lucas2015.h.lst[which(!lucas2015.h.lst %in% names(lucas2015.xy))]
  if(length(x.na)>0){ for(i in x.na){ lucas2015.xy[,i] = NA } }
  chemsprops.LUCAS2 <- lucas2015.xy[,lucas2015.h.lst]
  chemsprops.LUCAS2$source_db = "LUCAS_2015"
  chemsprops.LUCAS2$hzn_top <- 0
  chemsprops.LUCAS2$hzn_bot <- 20
  chemsprops.LUCAS2$site_obsdate <- "2015"
  chemsprops.LUCAS2$confidence_degree = 2
  chemsprops.LUCAS2$project_url = "https://esdac.jrc.ec.europa.eu/"
  chemsprops.LUCAS2$citation_url = "https://doi.org/10.1111/ejss.12499"
  chemsprops.LUCAS2 = complete.vars(chemsprops.LUCAS2, sel = c("OC","Clay","pH_H20"), coords = c("coords.x1", "coords.x2"))
}
dim(chemsprops.LUCAS2)
#> [1] 21859    36
```


#### Mangrove forest soil DB

- Sanderman, J., Hengl, T., Fiske, G., Solvik, K., Adame, M. F., Benson, L., ... & Duncan, C. (2018). [A global map of mangrove forest soil carbon at 30 m spatial resolution](https://doi.org/10.1088/1748-9326/aabe1c). Environmental Research Letters, 13(5), 055002. Data download URL: https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/OCYUIT


```r
if(!exists("chemsprops.Mangroves")){
  mng.profs <- read.csv("/mnt/diskstation/data/Soil_points/INT/TNC_mangroves/mangrove_soc_database_v10_sites.csv", skip=1)
  mng.hors <- read.csv("/mnt/diskstation/data/Soil_points/INT/TNC_mangroves/mangrove_soc_database_v10_horizons.csv", skip=1)
  mngALL = plyr::join(mng.hors, mng.profs, by=c("Site.name"))
  mngALL$oc = mngALL$OC_final * 10
  mngALL$oc_d = mngALL$CD_calc * 1000
  mngALL$hzn_top = mngALL$U_depth * 100
  mngALL$hzn_bot = mngALL$L_depth * 100
  mngALL$wpg2 = 0
  #summary(mngALL$BD_reported) ## some very high values 3.26 t/m3
  mngALL$Year = ifelse(is.na(mngALL$Year_sampled), mngALL$Years_collected, mngALL$Year_sampled)
  mng.col = c("Site.name", "Site..", "Year", "Longitude_Adjusted", "Latitude_Adjusted", "labsampnum", "layer_sequence","hzn_top","hzn_bot","hzn_desgn", "tex_psda", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "wpg2", "BD_reported", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")  
  x.na = mng.col[which(!mng.col %in% names(mngALL))]
  if(length(x.na)>0){ for(i in x.na){ mngALL[,i] = NA } }
  chemsprops.Mangroves = mngALL[,mng.col]
  chemsprops.Mangroves$source_db = "MangrovesDB"
  chemsprops.Mangroves$confidence_degree = 4
  chemsprops.Mangroves$project_url = "http://maps.oceanwealth.org/mangrove-restoration/"
  chemsprops.Mangroves$citation_url = "https://doi.org/10.1088/1748-9326/aabe1c"
  chemsprops.Mangroves = complete.vars(chemsprops.Mangroves, sel = c("oc","BD_reported"), coords = c("Longitude_Adjusted", "Latitude_Adjusted"))
  #head(chemsprops.Mangroves)
  #levels(as.factor(mngALL$OK.to.release.))
  mng.rm = chemsprops.Mangroves$Site.name[chemsprops.Mangroves$Site.name %in% mngALL$Site.name[grep("N", mngALL$OK.to.release., ignore.case = FALSE)]]
}
dim(chemsprops.Mangroves)
#> [1] 7734   36
```

#### CIFOR peatland points

Peatland soil measurements (points) from the literature described in:

- Murdiyarso, D., Roman-Cuesta, R. M., Verchot, L. V., Herold, M., Gumbricht, T., Herold, N., & Martius, C. (2017). New map reveals more peat in the tropics (Vol. 189). CIFOR. https://doi.org/10.17528/cifor/006452


```r
if(!exists("chemsprops.Peatlands")){
  cif.hors <- read.csv("/mnt/diskstation/data/Soil_points/INT/CIFOR_peatlands/SOC_literature_CIFOR.csv")
  #summary(cif.hors$BD..g.cm..)
  #summary(cif.hors$SOC)
  cif.hors$oc = cif.hors$SOC * 10
  cif.hors$wpg2 = 0
  cif.hors$c_tot = cif.hors$TOC.content.... * 10
  cif.hors$oc_d = cif.hors$C.density..kg.C.m..
  cif.hors$site_obsdate = as.integer(substr(cif.hors$year, 1, 4))-1
  cif.col = c("SOURCEID", "usiteid", "site_obsdate", "modelling.x", "modelling.y", "labsampnum", "layer_sequence", "Upper", "Lower", "hzn_desgn", "tex_psda", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "wpg2", "BD..g.cm..", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = cif.col[which(!cif.col %in% names(cif.hors))]
  if(length(x.na)>0){ for(i in x.na){ cif.hors[,i] = NA } }
  chemsprops.Peatlands = cif.hors[,cif.col]
  chemsprops.Peatlands$source_db = "CIFOR"
  chemsprops.Peatlands$confidence_degree = 4
  chemsprops.Peatlands$project_url = "https://www.cifor.org/"
  chemsprops.Peatlands$citation_url = "https://doi.org/10.17528/cifor/006452"
  chemsprops.Peatlands = complete.vars(chemsprops.Peatlands, sel = c("oc","BD..g.cm.."), coords = c("modelling.x", "modelling.y"))  
}
dim(chemsprops.Peatlands)
#> [1] 756  36
```


#### LandPKS observations

- Herrick, J. E., Urama, K. C., Karl, J. W., Boos, J., Johnson, M. V. V., Shepherd, K. D., ... & Kosnik, C. (2013). [The Global Land-Potential Knowledge System (LandPKS): Supporting Evidence-based, Site-specific Land Use and Management through Cloud Computing, Mobile Applications, and Crowdsourcing](https://doi.org/10.2489/jswc.68.1.5A). Journal of Soil and Water Conservation, 68(1), 5A-12A. Data download URL: http://portal.landpotential.org/#/landpksmap 


```r
if(!exists("chemsprops.LandPKS")){
  pks = read.csv("/mnt/diskstation/data/Soil_points/INT/LandPKS/Export_LandInfo_Data.csv", stringsAsFactors = FALSE)
  #str(pks)
  pks.hor = data.frame(rock_fragments = c(pks$rock_fragments_layer_0_1cm,
                                          pks$rock_fragments_layer_1_10cm,
                                          pks$rock_fragments_layer_10_20cm,
                                          pks$rock_fragments_layer_20_50cm,
                                          pks$rock_fragments_layer_50_70cm,
                                          pks$rock_fragments_layer_70_100cm,
                                          pks$rock_fragments_layer_100_120cm), 
                       tex_field = c(pks$texture_layer_0_1cm, 
                                     pks$texture_layer_1_10cm, 
                                     pks$texture_layer_10_20cm, 
                                     pks$texture_layer_20_50cm, 
                                     pks$texture_layer_50_70cm, 
                                     pks$texture_layer_70_100cm, 
                                     pks$texture_layer_100_120cm))
  pks.hor$hzn_top = c(rep(0, nrow(pks)), 
                      rep(1, nrow(pks)), 
                      rep(10, nrow(pks)), 
                      rep(20, nrow(pks)), 
                      rep(50, nrow(pks)), 
                      rep(70, nrow(pks)), 
                      rep(100, nrow(pks)))
  pks.hor$hzn_bot = c(rep(1, nrow(pks)), 
                      rep(10, nrow(pks)), 
                      rep(20, nrow(pks)), 
                      rep(50, nrow(pks)), 
                      rep(70, nrow(pks)), 
                      rep(100, nrow(pks)), 
                      rep(120, nrow(pks)))
  pks.hor$longitude_decimal_degrees = rep(pks$longitude, 7)
  pks.hor$latitude_decimal_degrees = rep(pks$latitude, 7)
  pks.hor$site_obsdate = rep(pks$modified_date, 7)
  pks.hor$site_key = rep(pks$id, 7)
  #summary(as.factor(pks.hor$tex_field))
  tex.tr = data.frame(tex_field=c("CLAY", "CLAY LOAM", "LOAM", "LOAMY SAND", "SAND", "SANDY CLAY", "SANDY CLAY LOAM", "SANDY LOAM", "SILT LOAM", "SILTY CLAY", "SILTY CLAY LOAM"), 
                      clay_tot_psa=c(62.4, 34.0, 19.0, 5.8, 3.3, 41.7, 27.0, 10.0, 13.1, 46.7, 34.0), 
                      silt_tot_psa=c(17.8, 34.0, 40.0, 12.0, 5.0, 6.7, 13.0, 25.0, 65.7, 46.7, 56.0), 
                      sand_tot_psa=c(19.8, 32.0, 41.0, 82.2, 91.7, 51.6, 60.0, 65.0, 21.2, 6.7, 10.0))
  pks.hor$clay_tot_psa = plyr::join(pks.hor["tex_field"], tex.tr)$clay_tot_psa
  pks.hor$silt_tot_psa = plyr::join(pks.hor["tex_field"], tex.tr)$silt_tot_psa
  pks.hor$sand_tot_psa = plyr::join(pks.hor["tex_field"], tex.tr)$sand_tot_psa
  #summary(as.factor(pks.hor$rock_fragments))
  pks.hor$wpg2 = ifelse(pks.hor$rock_fragments==">60%", 65, ifelse(pks.hor$rock_fragments=="35-60%", 47.5, ifelse(pks.hor$rock_fragments=="15-35%", 25, ifelse(pks.hor$rock_fragments=="1-15%" | pks.hor$rock_fragments=="0-15%", 7.5, ifelse(pks.hor$rock_fragments=="0-1%", 0.5, NA)))))
  #head(pks.hor)
  #plot(pks.hor[,c("longitude_decimal_degrees","latitude_decimal_degrees")])
  pks.col = c("site_key", "usiteid", "site_obsdate", "longitude_decimal_degrees", "latitude_decimal_degrees", "labsampnum", "layer_sequence","hzn_top","hzn_bot","hzn_desgn", "tex_field", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "wpg2", "db_od", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = pks.col[which(!pks.col %in% names(pks.hor))]
  if(length(x.na)>0){ for(i in x.na){ pks.hor[,i] = NA } }
  chemsprops.LandPKS = pks.hor[,pks.col]
  chemsprops.LandPKS$source_db = "LandPKS"
  chemsprops.LandPKS$confidence_degree = 8
  chemsprops.LandPKS$project_url = "http://portal.landpotential.org"
  chemsprops.LandPKS$citation_url = "https://doi.org/10.2489/jswc.68.1.5A"
  chemsprops.LandPKS = complete.vars(chemsprops.LandPKS, sel = c("clay_tot_psa","wpg2"), coords = c("longitude_decimal_degrees", "latitude_decimal_degrees"))
}
dim(chemsprops.LandPKS)
#> [1] 41644    36
```


#### EGRPR

- [Russian Federation: The Unified State Register of Soil Resources (EGRPR)](http://egrpr.esoil.ru/). Data download URL: http://egrpr.esoil.ru/content/1DB.html


```r
if(!exists("chemsprops.EGRPR")){
  russ.HOR = read.csv("/mnt/diskstation/data/Soil_points/Russia/EGRPR/Russia_EGRPR_soil_pedons.csv")
  russ.HOR$SOURCEID = paste(russ.HOR$CardID, russ.HOR$SOIL_ID, sep="_")
  russ.HOR$wpg2 = russ.HOR$TEXTSTNS
  russ.HOR$SNDPPT <- russ.HOR$TEXTSAF + russ.HOR$TEXSCM
  russ.HOR$SLTPPT <- russ.HOR$TEXTSIC + russ.HOR$TEXTSIM + 0.8 * ifelse(is.na(russ.HOR$TEXTSIF), 0, russ.HOR$TEXTSIF)
  russ.HOR$CLYPPT <- russ.HOR$TEXTCL + 0.2 * ifelse(is.na(russ.HOR$TEXTSIF), 0, russ.HOR$TEXTSIF)
  ## Correct texture fractions:
  sumTex <- rowSums(russ.HOR[,c("SLTPPT","CLYPPT","SNDPPT")])
  russ.HOR$SNDPPT <- russ.HOR$SNDPPT / ((sumTex - russ.HOR$CLYPPT) /(100 - russ.HOR$CLYPPT))
  russ.HOR$SLTPPT <- russ.HOR$SLTPPT / ((sumTex - russ.HOR$CLYPPT) /(100 - russ.HOR$CLYPPT))
  russ.HOR$oc <- rowMeans(data.frame(x1=russ.HOR$CORG * 10, x2=russ.HOR$ORGMAT/1.724 * 10), na.rm=TRUE)
  russ.HOR$oc_d = signif(russ.HOR$oc / 1000 * russ.HOR$DVOL * 1000 * (100 - ifelse(is.na(russ.HOR$wpg2), 0, russ.HOR$wpg2))/100, 3)
  russ.HOR$n_tot <- russ.HOR$NTOT * 10
  russ.HOR$ca_ext = russ.HOR$EXCA * 200
  russ.HOR$mg_ext = russ.HOR$EXMG * 121
  russ.HOR$na_ext = russ.HOR$EXNA * 230
  russ.HOR$k_ext = russ.HOR$EXK * 391
  ## Sampling year not available but with high confidence <2000
  russ.HOR$site_obsdate = "1982"
  russ.sel.h <- c("SOURCEID", "SOIL_ID", "site_obsdate", "LONG", "LAT", "labsampnum", "HORNMB", "HORTOP", "HORBOT", "HISMMN", "tex_psda", "CLYPPT", "SLTPPT", "SNDPPT", "oc", "oc_d", "c_tot", "NTOT", "PHSLT", "PHH2O", "ph_cacl2", "CECST", "cec_nh4", "ecec", "wpg2", "DVOL", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = russ.sel.h[which(!russ.sel.h %in% names(russ.HOR))]
  if(length(x.na)>0){ for(i in x.na){ russ.HOR[,i] = NA } }
  chemsprops.EGRPR = russ.HOR[,russ.sel.h]
  chemsprops.EGRPR$source_db = "Russia_EGRPR"
  chemsprops.EGRPR$confidence_degree = 2
  chemsprops.EGRPR$project_url = "http://egrpr.esoil.ru/"
  chemsprops.EGRPR$citation_url = "https://doi.org/10.19047/0136-1694-2016-86-115-123"
  chemsprops.EGRPR <- complete.vars(chemsprops.EGRPR, sel=c("oc", "CLYPPT"), coords = c("LONG", "LAT"))
}
dim(chemsprops.EGRPR)
#> [1] 4437   36
```


#### Canada National Pedon DB

- [Agriculture and Agri-Food Canada National Pedon Database](https://open.canada.ca/data/en/dataset/6457fad6-b6f5-47a3-9bd1-ad14aea4b9e0). Data download URL: https://open.canada.ca/data/en/


```r
if(!exists("chemsprops.NPDB")){
  NPDB.nm = c("NPDB_V2_sum_source_info.csv","NPDB_V2_sum_chemical.csv", "NPDB_V2_sum_horizons_raw.csv", "NPDB_V2_sum_physical.csv")
  NPDB.HOR = plyr::join_all(lapply(paste0("/mnt/diskstation/data/Soil_points/Canada/NPDB/", NPDB.nm), read.csv), type = "full")
  NPDB.HOR$HISMMN = paste0(NPDB.HOR$HZN_MAS, NPDB.HOR$HZN_SUF, NPDB.HOR$HZN_MOD)
  NPDB.HOR$CARB_ORG[NPDB.HOR$CARB_ORG==9] <- NA
  NPDB.HOR$N_TOTAL[NPDB.HOR$N_TOTAL==9] <- NA
  NPDB.HOR$oc = NPDB.HOR$CARB_ORG * 10
  NPDB.HOR$oc_d = signif(NPDB.HOR$oc / 1000 * NPDB.HOR$BULK_DEN * 1000 * (100 - ifelse(is.na(NPDB.HOR$VC_SAND), 0, NPDB.HOR$VC_SAND))/100, 3)
  NPDB.HOR$ca_ext = NPDB.HOR$EXCH_CA * 200
  NPDB.HOR$mg_ext = NPDB.HOR$EXCH_MG * 121
  NPDB.HOR$na_ext = NPDB.HOR$EXCH_NA * 230
  NPDB.HOR$k_ext = NPDB.HOR$EXCH_K * 391
  npdb.sel.h = c("PEDON_ID", "usiteid", "CAL_YEAR", "DD_LONG", "DD_LAT", "labsampnum", "layer_sequence", "U_DEPTH", "L_DEPTH", "HISMMN", "tex_psda", "T_CLAY", "T_SILT", "T_SAND", "oc", "oc_d", "c_tot", "N_TOTAL", "ph_kcl", "PH_H2O", "PH_CACL2", "CEC", "cec_nh4", "ecec", "VC_SAND", "BULK_DEN", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = npdb.sel.h[which(!npdb.sel.h %in% names(NPDB.HOR))]
  if(length(x.na)>0){ for(i in x.na){ NPDB.HOR[,i] = NA } }
  chemsprops.NPDB = NPDB.HOR[,npdb.sel.h]
  chemsprops.NPDB$source_db = "Canada_NPDB"
  chemsprops.NPDB$confidence_degree = 2
  chemsprops.NPDB$project_url = "https://open.canada.ca/data/en/"
  chemsprops.NPDB$citation_url = "https://open.canada.ca/data/en/dataset/6457fad6-b6f5-47a3-9bd1-ad14aea4b9e0"
  chemsprops.NPDB <- complete.vars(chemsprops.NPDB, sel=c("oc", "PH_H2O", "T_CLAY"), coords = c("DD_LONG", "DD_LAT"))
}
dim(chemsprops.NPDB)
#> [1] 15946    36
```


#### Canadian upland forest soil profile and carbon stocks database

- Shaw, C., Hilger, A., Filiatrault, M., & Kurz, W. (2018). [A Canadian upland forest soil profile and carbon stocks database](https://doi.org/10.1002/ecy.2159). Ecology, 99(4), 989-989. Data download URL: https://esajournals.onlinelibrary.wiley.com/action/downloadSupplement?doi=10.1002%2Fecy.2159&file=ecy2159-sup-0001-DataS1.zip 

*Organic horizons have negative values, the first mineral soil horizon has a value of 0 cm, and other mineral soil horizons have positive values. This needs to be corrected before the values can be bind with other international sets.


```r
if(!exists("chemsprops.CUFS")){
  ## Reading of the .dat file was tricky 
  cufs.HOR = read.csv("/mnt/diskstation/data/Soil_points/Canada/CUFSDB/PROFILES.csv", stringsAsFactors = FALSE)
  cufs.HOR$LOWER_HZN_LIMIT =cufs.HOR$UPPER_HZN_LIMIT + cufs.HOR$HZN_THICKNESS
  ## Correct depth (Canadian data can have negative depths for soil horizons):
  z.min.cufs <- ddply(cufs.HOR, .(LOCATION_ID), summarize, aggregated = min(UPPER_HZN_LIMIT, na.rm=TRUE))
  z.shift.cufs <- join(cufs.HOR["LOCATION_ID"], z.min.cufs, type="left")$aggregated
  ## fixed shift
  z.shift.cufs <- ifelse(z.shift.cufs>0, 0, z.shift.cufs)
  cufs.HOR$hzn_top <- cufs.HOR$UPPER_HZN_LIMIT - z.shift.cufs
  cufs.HOR$hzn_bot <- cufs.HOR$LOWER_HZN_LIMIT - z.shift.cufs
  cufs.SITE = read.csv("/mnt/diskstation/data/Soil_points/Canada/CUFSDB/SITES.csv", stringsAsFactors = FALSE)
  cufs.HOR$longitude_decimal_degrees = plyr::join(cufs.HOR["LOCATION_ID"], cufs.SITE)$LONGITUDE
  cufs.HOR$latitude_decimal_degrees = plyr::join(cufs.HOR["LOCATION_ID"], cufs.SITE)$LATITUDE
  cufs.HOR$site_obsdate = plyr::join(cufs.HOR["LOCATION_ID"], cufs.SITE)$YEAR_SAMPLED
  cufs.HOR$usiteid = plyr::join(cufs.HOR["LOCATION_ID"], cufs.SITE)$RELEASE_SOURCE_SITEID
  #summary(cufs.HOR$ORG_CARB_PCT)
  #hist(cufs.HOR$ORG_CARB_PCT, breaks=45)
  cufs.HOR$oc = cufs.HOR$ORG_CARB_PCT*10
  #cufs.HOR$c_tot = cufs.HOR$oc + ifelse(is.na(cufs.HOR$CARBONATE_CARB_PCT), 0, cufs.HOR$CARBONATE_CARB_PCT*10)
  cufs.HOR$n_tot = cufs.HOR$TOT_NITRO_PCT*10
  cufs.HOR$ca_ext = cufs.HOR$EXCH_Ca * 200
  cufs.HOR$mg_ext = cufs.HOR$EXCH_Mg * 121
  cufs.HOR$na_ext = cufs.HOR$EXCH_Na * 230
  cufs.HOR$k_ext = cufs.HOR$EXCH_K * 391
  cufs.HOR$ph_cacl2 = cufs.HOR$pH
  cufs.HOR$ph_cacl2[!cufs.HOR$pH_H2O_CACL2=="CACL2"] = NA
  cufs.HOR$ph_h2o = cufs.HOR$pH
  cufs.HOR$ph_h2o[!cufs.HOR$pH_H2O_CACL2=="H2O"] = NA
  #summary(cufs.HOR$CF_VOL_PCT) ## is NA == 0??
  cufs.HOR$wpg2 = ifelse(cufs.HOR$CF_CORR_FACTOR==1, 0, cufs.HOR$CF_VOL_PCT)
  cufs.HOR$oc_d = signif(cufs.HOR$oc / 1000 * cufs.HOR$BULK_DENSITY * 1000 * (100 - ifelse(is.na(cufs.HOR$wpg2), 0, cufs.HOR$wpg2))/100, 3)
  cufs.sel.h = c("LOCATION_ID", "usiteid", "site_obsdate", "longitude_decimal_degrees", "latitude_decimal_degrees", "labsampnum", "HZN_SEQ_NO", "hzn_top", "hzn_bot", "HORIZON", "TEXT_CLASS", "CLAY_PCT", "SILT_PCT", "SAND_PCT", "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "ph_cacl2", "CEC_CALCULATED", "cec_nh4", "ecec", "wpg2", "BULK_DENSITY", "ca_ext", "mg_ext", "na_ext", "k_ext", "ELEC_COND", "ec_12pre")
  x.na = cufs.sel.h[which(!cufs.sel.h %in% names(cufs.HOR))]
  if(length(x.na)>0){ for(i in x.na){ cufs.HOR[,i] = NA } }
  chemsprops.CUFS = cufs.HOR[,cufs.sel.h]
  chemsprops.CUFS$source_db = "Canada_CUFS"
  chemsprops.CUFS$confidence_degree = 1
  chemsprops.CUFS$project_url = "https://cfs.nrcan.gc.ca/publications/centre/nofc"
  chemsprops.CUFS$citation_url = "https://doi.org/10.1002/ecy.2159"
  chemsprops.CUFS <- complete.vars(chemsprops.CUFS, sel=c("oc", "ph_h2o", "CLAY_PCT"))
}
dim(chemsprops.CUFS)
#> [1] 15162    36
```

#### Permafrost in subarctic Canada 

- Estop-Aragones, C.; Fisher, J.P.; Cooper, M.A.; Thierry, A.; Treharne, R.; Murton, J.B.; Phoenix, G.K.; Charman, D.J.; Williams, M.; Hartley, I.P. (2016). Bulk density, carbon and nitrogen content in soil profiles from permafrost in subarctic Canada. NERC Environmental Information Data Centre. <https://doi.org/10.5285/efa2a84b-3505-4221-a7da-12af3cdc1952>. Data download URL:  


```r
if(!exists("chemsprops.CAPERM")){
  caperm.HOR = vroom::vroom("/mnt/diskstation/data/Soil_points/Canada/NorthCanada/Bulk_density_CandNcontent_profiles_all_sites.csv")
  #measurements::conv_unit("-99 36 15.7", from = "deg_min_sec", to = "dec_deg")
  #caperm.HOR$longitude_decimal_degrees = as.numeric(measurements::conv_unit(paste0("-", gsub('\"W', '', gsub("'", ' ', iconv(caperm.HOR$Coordinates_West, "UTF-8", "UTF-8", sub=' ')), fixed = TRUE)), from = "deg_min_sec", to = "dec_deg"))
  caperm.HOR$longitude_decimal_degrees = as.numeric(measurements::conv_unit(paste0("-", caperm.HOR$Cordinates_West), from = "deg_min_sec", to = "dec_deg"))
  #caperm.HOR$latitude_decimal_degrees = as.numeric(measurements::conv_unit(gsub('\"N', '', gsub('o', '', gsub("'", ' ', iconv(caperm.HOR$Coordinates_North, "UTF-8", "UTF-8", sub=' '))), fixed = TRUE), from = "deg_min_sec", to = "dec_deg"))
  caperm.HOR$latitude_decimal_degrees = as.numeric(measurements::conv_unit(caperm.HOR$Cordinates_North, from = "deg_min_sec", to = "dec_deg"))
  #plot(caperm.HOR[,c("longitude_decimal_degrees","latitude_decimal_degrees")])
  caperm.HOR$site_obsdate = "2013"
  caperm.HOR$site_key = make.unique(caperm.HOR$Soil.core)
  #summary(as.factor(caperm.HOR$Soil_depth_cm))
  caperm.HOR$hzn_top = caperm.HOR$Soil_depth_cm-1
  caperm.HOR$hzn_bot = caperm.HOR$Soil_depth_cm+1
  caperm.HOR$db_od = caperm.HOR$Bulk_density_gdrysoil_cm3wetsoil
  caperm.HOR$oc = caperm.HOR$Ccontent_percentage_on_drymass * 10
  caperm.HOR$n_tot = caperm.HOR$Ncontent_percentage_on_drymass * 10
  caperm.HOR$oc_d = signif(caperm.HOR$oc / 1000 * caperm.HOR$db_od * 1000, 3)
  x.na = col.names[which(!col.names %in% names(caperm.HOR))]
  if(length(x.na)>0){ for(i in x.na){ caperm.HOR[,i] = NA } }
  chemsprops.CAPERM = caperm.HOR[,col.names]
  chemsprops.CAPERM$source_db = "Canada_subarctic"
  chemsprops.CAPERM$confidence_degree = 2
  chemsprops.CAPERM$project_url = "http://arp.arctic.ac.uk/projects/carbon-cycling-linkages-permafrost-systems-cyclops/"
  chemsprops.CAPERM$citation_url = "https://doi.org/10.5285/efa2a84b-3505-4221-a7da-12af3cdc1952"
  chemsprops.CAPERM <- complete.vars(chemsprops.CAPERM, sel=c("oc", "n_tot"))
}
dim(chemsprops.CAPERM)
#> [1] 1180   36
```

#### SOTER China soil profiles

- Dijkshoorn, K., van Engelen, V., & Huting, J. (2008). [Soil and landform properties for LADA partner countries](https://isric.org/sites/default/files/isric_report_2008_06.pdf). ISRIC report 2008/06 and GLADA report 2008/03, ISRIC – World Soil Information and FAO, Wageningen. Data download URL: https://files.isric.org/public/soter/CN-SOTER.zip


```r
if(!exists("chemsprops.CNSOTER")){
  sot.sites = read.csv("/mnt/diskstation/data/Soil_points/China/China_SOTERv1/CHINA_SOTERv1_Profile.csv")
  sot.horizons = read.csv("/mnt/diskstation/data/Soil_points/China/China_SOTERv1/CHINA_SOTERv1_Horizon.csv")
  sot.HOR = plyr::join_all(list(sot.sites, sot.horizons), type = "full")
  sot.HOR$oc = sot.HOR$SOCA * 10
  sot.HOR$ca_ext = sot.HOR$EXCA * 200
  sot.HOR$mg_ext = sot.HOR$EXMG * 121
  sot.HOR$na_ext = sot.HOR$EXNA * 230
  sot.HOR$k_ext = sot.HOR$EXCK * 391
  ## upper depth missing needs to be derived manually
  sot.HOR$hzn_top = NA
  sot.HOR$hzn_top[2:nrow(sot.HOR)] <- sot.HOR$HBDE[1:(nrow(sot.HOR)-1)]
  sot.HOR$hzn_top <- ifelse(sot.HOR$hzn_top > sot.HOR$HBDE, 0, sot.HOR$hzn_top)
  sot.HOR$hzn_top <- ifelse(sot.HOR$HONU==1 & is.na(sot.HOR$hzn_top), 0, sot.HOR$hzn_top)
  sot.HOR$oc_d = signif(sot.HOR$oc / 1000 * sot.HOR$BULK * 1000 * (100 - ifelse(is.na(sot.HOR$SDVC), 0, sot.HOR$SDVC))/100, 3)
  sot.sel.h = c("PRID", "PDID", "SAYR", "LNGI", "LATI", "labsampnum", "HONU", "hzn_top","HBDE","HODE", "PSCL", "CLPC", "STPC", "SDTO", "oc", "oc_d", "TOTC", "TOTN", "PHKC", "PHAQ", "ph_cacl2", "CECS", "cec_nh4", "ecec", "SDVC", "BULK", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = sot.sel.h[which(!sot.sel.h %in% names(sot.HOR))]
  if(length(x.na)>0){ for(i in x.na){ sot.HOR[,i] = NA } }
  chemsprops.CNSOT = sot.HOR[,sot.sel.h]
  chemsprops.CNSOT$source_db = "China_SOTER"
  chemsprops.CNSOT$confidence_degree = 8
  chemsprops.CNSOT$project_url = "https://www.isric.org/explore/soter"
  chemsprops.CNSOT$citation_url = "https://isric.org/sites/default/files/isric_report_2008_06.pdf"
  chemsprops.CNSOT <- complete.vars(chemsprops.CNSOT, sel=c("TOTC", "PHAQ", "CLPC"), coords = c("LNGI", "LATI"))
}
#> Joining by: PRID, INFR
dim(chemsprops.CNSOT)
#> [1] 5105   36
```


#### SISLAC

- Sistema de Información de Suelos de Latinoamérica (SISLAC), Data download URL: <http://54.229.242.119/sislac/es>


```r
if(!exists("chemsprops.SISLAC")){
  sis.hor = read.csv("/mnt/diskstation/data/Soil_points/SA/SISLAC/sislac_profiles_es.csv", stringsAsFactors = FALSE)
  #str(sis.hor)
  ## SOC for Urugvay do not match the original soil profile data (see e.g. http://www.mgap.gub.uy/sites/default/files/multimedia/skmbt_c45111090914030.pdf)
  ## compare with:
  #sis.hor[sis.hor$perfil_id=="23861",]
  ## Subset to SISINTA/WOSIS points:
  cor.sel = c(grep("WoSIS", paste(sis.hor$perfil_numero)), grep("SISINTA", paste(sis.hor$perfil_numero)))
  #length(cor.sel)
  sis.hor = sis.hor[cor.sel,]
  #summary(sis.hor$analitico_carbono_organico_c)
  sis.hor$oc = sis.hor$analitico_carbono_organico_c * 10
  sis.hor$oc_d = signif(sis.hor$oc / 1000 * sis.hor$analitico_densidad_aparente * 1000 * (100 - ifelse(is.na(sis.hor$analitico_gravas), 0, sis.hor$analitico_gravas))/100, 3)
  #summary(sis.hor$analitico_base_k)
  #summary(as.factor(sis.hor$perfil_fecha))
  sis.sel.h = c("perfil_id", "perfil_numero", "perfil_fecha", "perfil_ubicacion_longitud", "perfil_ubicacion_latitud", "id", "layer_sequence", "profundidad_superior", "profundidad_inferior", "hzn_desgn", "tex_psda", "analitico_arcilla", "analitico_limo_2_50", "analitico_arena_total", "oc", "oc_d", "c_tot", "n_tot", "analitico_ph_kcl", "analitico_ph_h2o", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "analitico_gravas", "analitico_densidad_aparente", "ca_ext", "mg_ext", "na_ext", "k_ext", "analitico_conductividad", "ec_12pre")
  x.na = sis.sel.h[which(!sis.sel.h %in% names(sis.hor))]
  if(length(x.na)>0){ for(i in x.na){ sis.hor[,i] = NA } }
  chemsprops.SISLAC = sis.hor[,sis.sel.h]
  chemsprops.SISLAC$source_db = "SISLAC"
  chemsprops.SISLAC$confidence_degree = 4
  chemsprops.SISLAC$project_url = "http://54.229.242.119/sislac/es"
  chemsprops.SISLAC$citation_url = "https://hdl.handle.net/10568/49611"
  chemsprops.SISLAC <- complete.vars(chemsprops.SISLAC, sel=c("oc","analitico_ph_kcl","analitico_arcilla"), coords = c("perfil_ubicacion_longitud", "perfil_ubicacion_latitud"))
}
dim(chemsprops.SISLAC)
#> [1] 49994    36
```


#### FEBR

- Samuel-Rosa, A., Dalmolin, R. S. D., Moura-Bueno, J. M., Teixeira, W. G., & Alba, J. M. F. (2020). Open legacy soil survey data in Brazil: geospatial data quality and how to improve it. Scientia Agricola, 77(1). https://doi.org/10.1590/1678-992x-2017-0430
- Free Brazilian Repository for Open Soil Data – febr. Data download URL: http://www.ufsm.br/febr/


```r
if(!exists("chemsprops.FEBR")){
  #library(febr)
  ## download up-to-date copy of data
  #febr.lab = febr::layer(dataset = "all", variable="all")
  #febr.lab = febr::observation(dataset = "all")
  febr.hor = read.csv("/mnt/diskstation/data/Soil_points/Brasil/FEBR/febr-superconjunto.csv", stringsAsFactors = FALSE, dec = ",", sep = ";")
  #head(febr.hor)
  #summary(febr.hor$carbono)
  #summary(febr.hor$ph)
  #summary(febr.hor$dsi) ## bulk density of total soil
  febr.hor$clay_tot_psa = febr.hor$argila /10
  febr.hor$sand_tot_psa = febr.hor$areia /10
  febr.hor$silt_tot_psa = febr.hor$silte /10
  febr.hor$wpg2 = (1000-febr.hor$terrafina)/10
  febr.hor$oc_d = signif(febr.hor$carbono / 1000 * febr.hor$dsi * 1000 * (100 - ifelse(is.na(febr.hor$wpg2), 0, febr.hor$wpg2))/100, 3)
  febr.sel.h <- c("observacao_id", "usiteid", "observacao_data", "coord_x", "coord_y", "sisb_id", "camada_id", "profund_sup", "profund_inf", "camada_nome", "tex_psda", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", "carbono", "oc_d", "c_tot", "nitrogenio", "ph_kcl", "ph", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "wpg2", "dsi", "ca_ext", "mg_ext", "na_ext", "k_ext", "ce", "ec_12pre")
  x.na = febr.sel.h[which(!febr.sel.h %in% names(febr.hor))]
  if(length(x.na)>0){ for(i in x.na){ febr.hor[,i] = NA } }
  chemsprops.FEBR = febr.hor[,febr.sel.h]
  chemsprops.FEBR$source_db = "FEBR"
  chemsprops.FEBR$confidence_degree = 4
  chemsprops.FEBR$project_url = "http://www.ufsm.br/febr/"
  chemsprops.FEBR$citation_url = "https://doi.org/10.1590/1678-992x-2017-0430"
  chemsprops.FEBR <- complete.vars(chemsprops.FEBR, sel=c("carbono","ph","clay_tot_psa","dsi"), coords = c("coord_x", "coord_y"))
}
dim(chemsprops.FEBR)
#> [1] 7842   36
```

#### PRONASOLOS

- POLIDORO, J., COELHO, M., CARVALHO FILHO, A. D., LUMBRERAS, J., de OLIVEIRA, A. P., VASQUES, G. D. M., ... & BREFIN, M. (2021). [Programa Nacional de Levantamento e Interpretação de Solos do Brasil (PronaSolos): diretrizes para implementação](https://www.infoteca.cnptia.embrapa.br/infoteca/handle/doc/1135056). Embrapa Solos-Documentos (INFOTECA-E).
- Download URL: http://geoinfo.cnps.embrapa.br/documents/3013/download


```r
if(!exists("chemsprops.PRONASOLOS")){
  pronas.hor = as.data.frame(sf::read_sf("/mnt/diskstation/data/Soil_points/Brasil/Pronasolos/Perfis_PronaSolos_20201202v2.shp"))
  ## 34,464 rows
  #head(pronas.hor)
  #summary(as.numeric(pronas.hor$carbono_or))
  #summary(as.numeric(pronas.hor$densidade_))
  #summary(as.numeric(pronas.hor$argila))
  #summary(as.numeric(pronas.hor$cascalho))
  #summary(as.numeric(pronas.hor$ph_h2o))
  #summary(as.numeric(pronas.hor$complexo_2))
  ## A lot of errors / typos e.g. very high values and 0 values!!
  #pronas.hor$data_colet[1:50]
  pronas.in.name = c("sigla", "codigo_pon", "data_colet", "gcs_latitu", "gcs_longit", "simbolo_ho", "profundida", 
                     "profundi_1", "cascalho", "areia_tota", "silte", "argila", "densidade_", "ph_h2o", "ph_kcl", 
                     "complexo_s", "complexo_1", "complexo_2", "complexo_3", "valor_s", "carbono_or", "nitrogenio",
                     "condutivid", "classe_tex")
  #pronas.in.name[which(!pronas.in.name %in% names(pronas.hor))]
  pronas.x = as.data.frame(pronas.hor[,pronas.in.name])
  pronas.out.name = c("site_key", "usiteid", "site_obsdate", "latitude_decimal_degrees", "longitude_decimal_degrees",
                      "hzn_desgn", "hzn_bot", "hzn_top", "wpg2", "sand_tot_psa", "silt_tot_psa", 
                      "clay_tot_psa", "db_od", "ph_h2o", "ph_kcl", "ca_ext",
                      "mg_ext", "k_ext", "na_ext", "cec_sum", "oc", "n_tot", "ec_satp", "tex_psda")
  ## translate values
  pronas.fun.lst = as.list(rep("as.numeric(x)*1", length(pronas.in.name)))
  pronas.fun.lst[[which(pronas.in.name=="sigla")]] = "paste(x)"
  pronas.fun.lst[[which(pronas.in.name=="codigo_pon")]] = "paste(x)"
  pronas.fun.lst[[which(pronas.in.name=="data_colet")]] = "paste(x)"
  pronas.fun.lst[[which(pronas.in.name=="simbolo_ho")]] = "paste(x)"
  pronas.fun.lst[[which(pronas.in.name=="classe_tex")]] = "paste(x)"
  pronas.fun.lst[[which(pronas.in.name=="complexo_s")]] = "as.numeric(x)*200"
  pronas.fun.lst[[which(pronas.in.name=="complexo_1")]] = "as.numeric(x)*121"
  pronas.fun.lst[[which(pronas.in.name=="complexo_2")]] = "as.numeric(x)*391"
  pronas.fun.lst[[which(pronas.in.name=="complexo_3")]] = "as.numeric(x)*230"
  pronas.fun.lst[[which(pronas.in.name=="areia_tota")]] = "round(as.numeric(x)/10, 1)"
  pronas.fun.lst[[which(pronas.in.name=="silte")]] = "round(as.numeric(x)/10, 1)"
  pronas.fun.lst[[which(pronas.in.name=="argila")]] = "round(as.numeric(x)/10, 1)"
  ## save translation rules:
  write.csv(data.frame(pronas.in.name, pronas.out.name, unlist(pronas.fun.lst)), "pronas_soilab_transvalues.csv")
  pronas.soil = transvalues(pronas.x, pronas.out.name, pronas.in.name, pronas.fun.lst)
  pronas.soil$oc_d = signif(pronas.soil$oc / 1000 * pronas.soil$db_od * 1000 * (100 - ifelse(is.na(pronas.soil$wpg2), 0, pronas.soil$wpg2))/100, 3)
  x.na = col.names[which(!col.names %in% names(pronas.soil))]
  if(length(x.na)>0){ for(i in x.na){ pronas.soil[,i] = NA } }
  chemsprops.PRONASOLOS = pronas.soil[,col.names]
  chemsprops.PRONASOLOS$source_db = "PRONASOLOS"
  chemsprops.PRONASOLOS$confidence_degree = 2
  chemsprops.PRONASOLOS$project_url = "https://geoportal.cprm.gov.br/pronasolos/"
  chemsprops.PRONASOLOS$citation_url = "https://www.infoteca.cnptia.embrapa.br/infoteca/handle/doc/1135056"
  chemsprops.PRONASOLOS <- complete.vars(chemsprops.PRONASOLOS, sel=c("oc","ph_h2o","clay_tot_psa"))
}
dim(chemsprops.PRONASOLOS)
#> [1] 31747    36
```


#### Soil Profile DB for Costa Rica

- Mata, R., Vázquez, A., Rosales, A., & Salazar, D. (2012). [Mapa digital de suelos de Costa Rica](http://www.cia.ucr.ac.cr/?page_id=139). Asociación Costarricense de la Ciencia del Suelo, San José, CRC. Escala, 1, 200000. Data download URL: http://www.cia.ucr.ac.cr/wp-content/recursosnaturales/Base%20perfiles%20de%20suelos%20v1.1.rar
- Mata-Chinchilla, R., & Castro-Chinchilla, J. (2019). Geoportal de suelos de Costa Rica como Bien Público al servicio del país. Revista Tecnología En Marcha, 32(7), Pág. 51-56. https://doi.org/10.18845/tm.v32i7.4259 


```r
if(!exists("chemsprops.CostaRica")){
  cr.hor = read.csv("/mnt/diskstation/data/Soil_points/Costa_Rica/Base_de_datos_version_1.2.3.csv", stringsAsFactors = FALSE)
  #plot(cr.hor[,c("X","Y")], pch="+", asp=1)
  cr.hor$usiteid = paste(cr.hor$Provincia, cr.hor$Cantón, cr.hor$Id, sep="_")
  #summary(cr.hor$Corg.)
  cr.hor$oc = cr.hor$Corg. * 10
  cr.hor$Densidad.Aparente = as.numeric(paste0(cr.hor$Densidad.Aparente))
  #summary(cr.hor$K)
  cr.hor$ca_ext = cr.hor$Ca * 200
  cr.hor$mg_ext = cr.hor$Mg * 121
  #cr.hor$na_ext = cr.hor$Na * 230
  cr.hor$k_ext = cr.hor$K * 391
  cr.hor$wpg2 = NA
  cr.hor$oc_d = signif(cr.hor$oc / 1000 * cr.hor$Densidad.Aparente * 1000 * (100 - ifelse(is.na(cr.hor$wpg2), 0, cr.hor$wpg2))/100, 3)
  cr.sel.h = c("Id", "usiteid", "Fecha", "X", "Y", "labsampnum", "horizonte", "prof_inicio", "prof_final", "id_hz", "Clase.Textural", "ARCILLA", "LIMO", "ARENA", "oc", "oc_d", "c_tot", "n_tot", "pHKCl", "pH_H2O", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "wpg2", "Densidad.Aparente", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = cr.sel.h[which(!cr.sel.h %in% names(cr.hor))]
  if(length(x.na)>0){ for(i in x.na){ cr.hor[,i] = NA } }
  chemsprops.CostaRica = cr.hor[,cr.sel.h]
  chemsprops.CostaRica$source_db = "CostaRica"
  chemsprops.CostaRica$confidence_degree = 4
  chemsprops.CostaRica$project_url = "http://www.cia.ucr.ac.cr"
  chemsprops.CostaRica$citation_url = "https://doi.org/10.18845/tm.v32i7.42"
  chemsprops.CostaRica <- complete.vars(chemsprops.CostaRica, sel=c("oc","pH_H2O","ARCILLA","Densidad.Aparente"), coords = c("X", "Y"))
}
dim(chemsprops.CostaRica)
#> [1] 2042   36
```


#### Iran soil profile DB

- Dewan, M. L., & Famouri, J. (1964). The soils of Iran. Food and Agriculture Organization of the United Nations.
- Hengl, T., Toomanian, N., Reuter, H. I., & Malakouti, M. J. (2007). [Methods to interpolate soil categorical variables from profile observations: Lessons from Iran](https://doi.org/10.1016/j.geoderma.2007.04.022). Geoderma, 140(4), 417-427.
- Mohammad, H. B. (2000). Soil resources and use potentiality map of Iran. Soil and Water Research Institute, Teheran, Iran.


```r
if(!exists("chemsprops.IRANSPDB")){
  na.s = c("?","","?.","??", -2147483647, -1.00e+308, "<NA>")
  iran.hor = read.csv("/mnt/diskstation/data/Soil_points/Iran/iran_sdbana.txt", stringsAsFactors = FALSE, na.strings = na.s, header = FALSE)[,1:12]
  names(iran.hor) = c("site_key", "hzn_desgn", "hzn_top", "hzn_bot", "ph_h2o", "ec_satp", "oc", "CACO", "PBS", "sand_tot_psa", "silt_tot_psa", "clay_tot_psa")
  iran.hor$hzn_top = ifelse(is.na(iran.hor$hzn_top) & iran.hor$hzn_desgn=="A", 0, iran.hor$hzn_top)
  iran.hor2 = read.csv("/mnt/diskstation/data/Soil_points/Iran/iran_sdbhor.txt", stringsAsFactors = FALSE, na.strings = na.s, header = FALSE)[,1:8]
  names(iran.hor2) = c("site_key", "layer_sequence", "DESI", "hzn_top", "hzn_bot", "M_colour", "tex_psda", "hzn_desgn")
  iran.site = read.csv("/mnt/diskstation/data/Soil_points/Iran/iran_sgdb.txt", stringsAsFactors = FALSE, na.strings = na.s, header = FALSE)
  names(iran.site) = c("usiteid", "latitude_decimal_degrees", "longitude_decimal_degrees", "FAO", "Tax", "site_key")
  iran.db = plyr::join_all(list(iran.site, iran.hor, iran.hor2))
  iran.db$oc = iran.db$oc * 10
  #summary(iran.db$oc)
  x.na = col.names[which(!col.names %in% names(iran.db))]
  if(length(x.na)>0){ for(i in x.na){ iran.db[,i] = NA } }
  chemsprops.IRANSPDB = iran.db[,col.names]
  chemsprops.IRANSPDB$source_db = "Iran_SPDB"
  chemsprops.IRANSPDB$confidence_degree = 4
  chemsprops.IRANSPDB$project_url = ""
  chemsprops.IRANSPDB$citation_url = "https://doi.org/10.1016/j.geoderma.2007.04.022"
  chemsprops.IRANSPDB <- complete.vars(chemsprops.IRANSPDB, sel=c("oc","ph_h2o","clay_tot_psa"))
}
dim(chemsprops.IRANSPDB)
#> [1] 4759   36
```

#### Northern circumpolar permafrost soil profiles

- Hugelius, G., Bockheim, J. G., Camill, P., Elberling, B., Grosse, G., Harden, J. W., ... & Michaelson, G. (2013). [A new data set for estimating organic carbon storage to 3 m depth in soils of the northern circumpolar permafrost region](https://doi.org/10.5194/essd-5-393-2013). Earth System Science Data (Online), 5(2). Data download URL: http://dx.doi.org/10.5879/ECDS/00000002


```r
if(!exists("chemsprops.NCSCD")){
  ncscd.hors <- read.csv("/mnt/diskstation/data/Soil_points/INT/NCSCD/Harden_etal_2012_Hugelius_etal_2013_cleaned_data.csv", stringsAsFactors = FALSE)
  ncscd.hors$oc = as.numeric(ncscd.hors$X.C)*10
  #summary(ncscd.hors$oc)
  #hist(ncscd.hors$Layer.thickness.cm, breaks = 45)
  ncscd.hors$Layer.thickness.cm = ifelse(ncscd.hors$Layer.thickness.cm<0, NA, ncscd.hors$Layer.thickness.cm)
  ncscd.hors$hzn_bot = ncscd.hors$Basal.Depth.cm + ncscd.hors$Layer.thickness.cm
  ncscd.hors$db_od = as.numeric(ncscd.hors$bulk.density.g.cm.3)
  ## Can we assume no coarse fragments?
  ncscd.hors$wpg2 = 0
  ncscd.hors$oc_d = signif(ncscd.hors$oc / 1000 * ncscd.hors$db_od * 1000 * (100 - ifelse(is.na(ncscd.hors$wpg2), 0, ncscd.hors$wpg2))/100, 3)
  ## very high values >40 kg/m3
  ncscd.hors$site_obsdate = format(as.Date(ncscd.hors$Sample.date, format="%d-%m-%Y"), "%Y-%m-%d")
  #summary(ncscd.hors$db_od)
  ncscd.col = c("Profile.ID", "citation", "site_obsdate", "Long", "Lat", "labsampnum", "layer_sequence", "Basal.Depth.cm", "hzn_bot", "Horizon.type", "tex_psda", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "wpg2", "db_od", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = ncscd.col[which(!ncscd.col %in% names(ncscd.hors))]
  if(length(x.na)>0){ for(i in x.na){ ncscd.hors[,i] = NA } }
  chemsprops.NCSCD = ncscd.hors[,ncscd.col]
  chemsprops.NCSCD$source_db = "NCSCD"
  chemsprops.NCSCD$confidence_degree = 10
  chemsprops.NCSCD$project_url = "https://bolin.su.se/data/ncscd/"
  chemsprops.NCSCD$citation_url = "https://doi.org/10.5194/essd-5-393-2013"
  chemsprops.NCSCD = complete.vars(chemsprops.NCSCD, sel = c("oc","db_od"), coords = c("Long", "Lat"))  
}
dim(chemsprops.NCSCD)
#> [1] 7104   36
```

#### CSIRO National Soil Site Database

- CSIRO (2020). CSIRO National Soil Site Database. v4. CSIRO. Data Collection. https://data.csiro.au/collections/collection/CIcsiro:7526v004. Data download URL: https://doi.org/10.25919/5eeb2a56eac12 (available upon request)
- Searle, R. (2014). The Australian site data collation to support the GlobalSoilMap. GlobalSoilMap: Basis of the global spatial soil information system, 127. 


```r
if(!exists("chemsprops.NatSoil")){
  library(Hmisc) 
  cmdb <- mdb.get("/mnt/diskstation/data/Soil_points/Australia/CSIRO/NatSoil_v2_20200612.mdb")
  #str(cmdb$SITES)
  au.obs = cmdb$OBSERVATIONS[,c("s.id", "o.location.notes", "o.date.desc", "o.latitude.GDA94", "o.longitude.GDA94")]
  au.obs = au.obs[!is.na(au.obs$o.longitude.GDA94),]
  coordinates(au.obs) <- ~o.longitude.GDA94+o.latitude.GDA94
  proj4string(au.obs) <- CRS("+proj=longlat +ellps=GRS80 +no_defs")
  au.xy <- data.frame(spTransform(au.obs, CRS("+proj=longlat +ellps=WGS84 +datum=WGS84")))
  #plot(au.xy[,c("o.longitude.GDA94", "o.latitude.GDA94")])
  ## all variables in one column and need to be sorted based on the lab method
  #summary(cmdb$LAB_METHODS$LABM.SHORT.NAME)
  #write.csv(cmdb$LAB_METHODS, "/mnt/diskstation/data/Soil_points/Australia/CSIRO/NatSoil_v2_20200612_lab_methods.csv")
  lab.tbl = list(
          c("6_DC", "6A1", "6A1_UC", "6B1", "6B2", "6B2a", "6B2b", "6B3", "6B4", "6B4a", "6B4b", "6Z"), # %
          c("6B3a"), # g/kg
          c("6H4", "6H4_SCaRP"), # %
          c("7_C_B", "7_NR", "7A1", "7A2", "7A2a", "7A2b", "7A3", "7A4", "7A5", "7A6", "7A6a", "7A6b", "7A6b_MCLW"),  # g/kg
          c("4A1", "4_NR", "4A_C_2.5", "4A_C_1", "4G1"),
          c("4C_C_1", "4C1", "4C2", "23A"),
          c("4B_C_2.5", "4B1", "4B2"),
          c("P10_NR_C", "P10_HYD_C", "P10_PB_C", "P10_PB1_C", "P10_CF_C", "P10_I_C"),
          c("P10_NR_Z", "P10_HYD_Z", "P10_PB_Z", "P10_PB1_Z", "P10_CF_Z", "P10_I_Z"),
          c("P10_NR_S", "P10_HYD_S", "P10_PB_S", "P10_PB1_S", "P10_CF_S", "P10_I_S"),
          c("15C1modCEC", "15_HSK_CEC", "15J_CEC"),
          c("15I1", "15I2", "15I3", "15I4", "15D3_CEC"),
          c("15_BASES", "15_NR", "15J_H", "15J1"),
          c("2Z2_Grav", "P10_GRAV"),
          c("503.08a", "P3A_NR", "P3A1", "P3A1_C4", "P3A1_CLOD", "P3A1_e"),
          c("18F1_CA"),
          c("18F1_MG"),
          c("18F1_NA"),
          c("18F1_K", "18F2", "18A1mod", "18_NR", "18A1", "18A1_NR", "18B1", "18B2"),
          c("3_C_B", "3_NR", "3A_TSS"),
          c("3A_C_2.5", "3A1")
          )
  names(lab.tbl) = c("oc", "ocP", "c_tot", "n_tot", "ph_h2o", "ph_kcl", "ph_cacl2", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", "cec_sum", "cec_nh4", "ecec", "wpg2", "db_od", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  val.lst = lapply(1:length(lab.tbl), function(i){x <- cmdb$LAB_RESULTS[cmdb$LAB_RESULTS$labm.code %in% lab.tbl[[i]], c("agency.code", "proj.code", "s.id", "o.id", "h.no", "labr.value")]; names(x)[6] <- names(lab.tbl)[i]; return(x) })
  names(val.lst) = names(lab.tbl)
  val.lst$oc$oc = val.lst$oc$oc * 10
  names(val.lst$ocP)[6] = "oc"
  val.lst$oc <- rbind(val.lst$oc, val.lst$ocP)
  val.lst$ocP = NULL
  #summary(val.lst$oc$oc)
  #str(val.lst, max.level = 1)
  for(i in 1:length(val.lst)){ val.lst[[i]]$h.id <- paste(val.lst[[i]]$agency.code, val.lst[[i]]$proj.code, val.lst[[i]]$s.id, val.lst[[i]]$o.id, val.lst[[i]]$h.no, sep="_") }
  au.hor <- plyr::join_all(lapply(val.lst, function(x){x[,6:7]}), match="first")
  #str(as.factor(au.hor$h.id))
  cmdb$HORIZONS$h.id = paste(cmdb$HORIZONS$agency.code, cmdb$HORIZONS$proj.code, cmdb$HORIZONS$s.id, cmdb$HORIZONS$o.id, cmdb$HORIZONS$h.no, sep="_")
  cmdb$HORIZONS$hzn_desgn = paste(cmdb$HORIZONS$h.desig.master, cmdb$HORIZONS$h.desig.subdiv, cmdb$HORIZONS$h.desig.suffix, sep="")
  au.horT <- plyr::join_all(list(cmdb$HORIZONS[,c("h.id","s.id","h.no","h.texture","hzn_desgn","h.upper.depth","h.lower.depth")], au.hor, au.xy))
  au.horT$site_obsdate = format(as.Date(au.horT$o.date.desc, format="%d%m%Y"), "%Y-%m-%d")
  au.horT$sand_tot_psa = ifelse(is.na(au.horT$sand_tot_psa), 100-(au.horT$clay_tot_psa + au.horT$silt_tot_psa), au.horT$sand_tot_psa)
  au.horT$hzn_top = au.horT$h.upper.depth*100
  au.horT$hzn_bot = au.horT$h.lower.depth*100
  au.horT$oc_d = signif(au.horT$oc / 1000 * au.horT$db_od * 1000 * (100 - ifelse(is.na(au.horT$wpg2), 0, au.horT$wpg2))/100, 3)
  au.cols.n = c("s.id", "o.location.notes", "site_obsdate", "o.longitude.GDA94", "o.latitude.GDA94", "h.id", "h.no", "hzn_top", "hzn_bot", "hzn_desgn", "h.texture", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "wpg2", "db_od", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = au.cols.n[which(!au.cols.n %in% names(au.horT))]
  if(length(x.na)>0){ for(i in x.na){ au.horT[,i] = NA } }
  chemsprops.NatSoil = au.horT[,au.cols.n]
  chemsprops.NatSoil$source_db = "CSIRO_NatSoil"
  chemsprops.NatSoil$confidence_degree = 4
  chemsprops.NatSoil$project_url = "https://www.csiro.au/en/Do-business/Services/Enviro/Soil-archive"
  chemsprops.NatSoil$citation_url = "https://doi.org/10.25919/5eeb2a56eac12"
  chemsprops.NatSoil = complete.vars(chemsprops.NatSoil, sel = c("oc","db_od","clay_tot_psa","ph_h2o"), coords = c("o.longitude.GDA94", "o.latitude.GDA94"))  
}
dim(chemsprops.NatSoil)
#> [1] 70791    36
```

#### NAMSOTER

- Coetzee, M. E. (2001). [NAMSOTER, a SOTER database for Namibia](https://edepot.wur.nl/485173). Agroecological Zoning, 458.
- Coetzee, M. E. (2009). Chemical characterisation of the soils of East Central Namibia (Doctoral dissertation, Stellenbosch: University of Stellenbosch).


```r
if(!exists("chemsprops.NAMSOTER")){
  nam.profs <- read.csv("/mnt/diskstation/data/Soil_points/Namibia/NAMSOTER/Namibia_all_profiles.csv", na.strings = c("-9999", "999", "9999", "NA"))
  nam.hors <- read.csv("/mnt/diskstation/data/Soil_points/Namibia/NAMSOTER/Namibia_all_horizons.csv", na.strings = c("-9999", "999", "9999", "NA"))
  #summary(nam.hors$TOTN)
  #summary(nam.hors$TOTC)
  nam.hors$hzn_top <- NA
  nam.hors$hzn_top <- ifelse(nam.hors$HONU==1, 0, nam.hors$hzn_top)
  h.lst <- lapply(1:7, function(x){which(nam.hors$HONU==x)})
  for(i in 2:7){
    sel <- match(nam.hors$PRID[h.lst[[i]]], nam.hors$PRID[h.lst[[i-1]]])
    nam.hors$hzn_top[h.lst[[i]]] <- nam.hors$HBDE[h.lst[[i-1]]][sel]
  }
  nam.hors$HBDE <- ifelse(is.na(nam.hors$HBDE), nam.hors$hzn_top+50, nam.hors$HBDE)
  #summary(nam.hors$HBDE)
  namALL = plyr::join(nam.hors, nam.profs, by=c("PRID"))
  namALL$k_ext = namALL$EXCK * 391
  namALL$ca_ext = namALL$EXCA * 200
  namALL$mg_ext = namALL$EXMG * 121
  namALL$na_ext = namALL$EXNA * 230
  #summary(namALL$MINA)
  namALL$BULK <- ifelse(namALL$BULK>2.4, NA, namALL$BULK)
  namALL$wpg2 = ifelse(namALL$MINA=="D", 80, ifelse(namALL$MINA=="A", 60, ifelse(namALL$MINA=="M", 25, ifelse(namALL$MINA=="C", 10, ifelse(namALL$MINA=="V", 1, ifelse(namALL$MINA=="F", 2.5, ifelse(namALL$MINA=="M/A", 40, ifelse(namALL$MINA=="C/M", 15, 0))))))))
  #hist(namALL$wpg2)
  namALL$oc_d = signif(namALL$TOTC / 1000 * namALL$BULK * 1000 * (100 - ifelse(is.na(namALL$wpg2), 0, namALL$wpg2))/100, 3)
  #summary(namALL$oc_d)
  #summary(namALL$PHAQ) ## very high ph
  namALL$site_obsdate = 2000
  nam.col = c("PRID", "SLID", "site_obsdate", "LONG", "LATI", "labsampnum", "HONU", "hzn_top", "HBDE", "HODE", "PSCL", "CLPC", "STPC", "SDTO", "TOTC", "oc_d", "c_tot", "TOTN", "PHKC", "PHAQ", "ph_cacl2", "CECS", "cec_nh4", "ecec", "wpg2", "BULK", "ca_ext", "mg_ext", "na_ext", "k_ext", "ELCO", "ec_12pre")
  x.na = nam.col[which(!nam.col %in% names(namALL))]
  if(length(x.na)>0){ for(i in x.na){ namALL[,i] = NA } }
  chemsprops.NAMSOTER = namALL[,nam.col]
  chemsprops.NAMSOTER$source_db = "NAMSOTER"
  chemsprops.NAMSOTER$confidence_degree = 2
  chemsprops.NAMSOTER$project_url = ""
  chemsprops.NAMSOTER$citation_url = "https://edepot.wur.nl/485173"
  chemsprops.NAMSOTER = complete.vars(chemsprops.NAMSOTER, sel = c("TOTC","CLPC","PHAQ"), coords = c("LONG", "LATI"))  
}
dim(chemsprops.NAMSOTER)
#> [1] 2953   36
```

#### Worldwide organic soil carbon and nitrogen data

- Zinke, P. J., Millemann, R. E., & Boden, T. A. (1986). [Worldwide organic soil carbon and nitrogen data](https://cdiac.ess-dive.lbl.gov/ftp/ndp018/ndp018.pdf). Carbon Dioxide Information Center, Environmental Sciences Division, Oak Ridge National Laboratory. Data download URL: https://dx.doi.org/10.3334/CDIAC/lue.ndp018 

- Note: poor spatial location accuracy i.e. <10 km. Bulk density for many points has been estimated not measured. Sampling year has not been but literature indicates: 1965, 1974, 1976, 1978, 1979, 1984. Most of samples come from natural vegetation (undisturbed) areas.


```r
if(!exists("chemsprops.ISCND")){
  ndp.profs <- read.csv("/mnt/diskstation/data/Soil_points/INT/ISCND/ndp018.csv", na.strings = c("-9999", "?", "NA"), stringsAsFactors = FALSE)
  names(ndp.profs) = c("PROFILE", "CODE", "CARBON", "NITROGEN", "LAT", "LONG", "ELEV", "SOURCE", "HOLDRIGE", "OLSON", "PARENT")
  for(j in c("CARBON","NITROGEN","ELEV")){  ndp.profs[,j] <- as.numeric(ndp.profs[,j])   }
  #summary(ndp.profs$CARBON)
  lat.s  <- grep("S", ndp.profs$LAT) # lat.n <- grep("N", ndp.profs$LAT)
  ndp.profs$latitude_decimal_degrees = as.numeric(gsub("[^0-9.-]", "", ndp.profs$LAT))
  ndp.profs$latitude_decimal_degrees[lat.s] = ndp.profs$latitude_decimal_degrees[lat.s] * -1
  lon.w <- grep("W", ndp.profs$LONG) # lon.e  <- grep("E", ndp.profs$LONG, fixed = TRUE)
  ndp.profs$longitude_decimal_degrees = as.numeric(gsub("[^0-9.-]", "", ndp.profs$LONG))
  ndp.profs$longitude_decimal_degrees[lon.w] = ndp.profs$longitude_decimal_degrees[lon.w] * -1
  #plot(ndp.profs[,c("longitude_decimal_degrees", "latitude_decimal_degrees")])
  ndp.profs$hzn_top = 0; ndp.profs$hzn_bot = 100
  ## Sampling years from the doc: 1965, 1974, 1976, 1978, 1979, 1984 
  ndp.profs$site_obsdate = "1982"
  ndp.col = c("PROFILE", "CODE", "site_obsdate", "longitude_decimal_degrees", "latitude_decimal_degrees", "labsampnum", "layer_sequence","hzn_top","hzn_bot","hzn_desgn", "tex_psda", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", "oc", "CARBON", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "wpg2", "db_od", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = ndp.col[which(!ndp.col %in% names(ndp.profs))]
  if(length(x.na)>0){ for(i in x.na){ ndp.profs[,i] = NA } }
  chemsprops.ISCND = ndp.profs[,ndp.col]
  chemsprops.ISCND$source_db = "ISCND"
  chemsprops.ISCND$confidence_degree = 8
  chemsprops.ISCND$project_url = "https://iscn.fluxdata.org/data/"
  chemsprops.ISCND$citation_url = "https://dx.doi.org/10.3334/CDIAC/lue.ndp018"
  chemsprops.ISCND = complete.vars(chemsprops.ISCND, sel = c("CARBON"), coords = c("longitude_decimal_degrees", "latitude_decimal_degrees"))  
}
dim(chemsprops.ISCND)
#> [1] 3977   36
```


#### Interior Alaska Carbon and Nitrogen stocks

- Manies, K., Waldrop, M., and Harden, J. (2020): Generalized models to estimate carbon and nitrogen stocks of organic soil horizons in Interior Alaska, Earth Syst. Sci. Data, 12, 1745–1757, https://doi.org/10.5194/essd-12-1745-2020, Data download URL: https://doi.org/10.5066/P960N1F9


```r
if(!exists("chemsprops.Alaska")){
  al.gps <- read.csv("/mnt/diskstation/data/Soil_points/USA/Alaska_Interior/Site_GPS_coordinates_v1-1.csv", stringsAsFactors = FALSE)
  ## Different datums!
  #summary(as.factor(al.gps$Datum))
  al.gps1 = al.gps[al.gps$Datum=="NAD83",]
  coordinates(al.gps1) = ~ Longitude + Latitude
  proj4string(al.gps1) = "+proj=longlat +datum=NAD83"
  al.gps0 = spTransform(al.gps1, CRS("+proj=longlat +datum=WGS84"))
  al.gps[which(al.gps$Datum=="NAD83"),"Longitude"] = al.gps0@coords[,1]
  al.gps[which(al.gps$Datum=="NAD83"),"Latitude"] = al.gps0@coords[,2]
  al.gps$site = al.gps$Site
  al.hor <- read.csv("/mnt/diskstation/data/Soil_points/USA/Alaska_Interior/Generalized_models_for_CandN_Alaska_v1-1.csv", stringsAsFactors = FALSE)
  al.hor$hzn_top = al.hor$depth - as.numeric(al.hor$thickness)
  al.hor$site_obsdate = format(as.Date(al.hor$date, format = "%m/%d/%Y"), "%Y-%m-%d")
  al.hor$oc = as.numeric(al.hor$carbon) * 10
  al.hor$n_tot = as.numeric(al.hor$nitrogen) * 10
  al.hor$oc_d = as.numeric(al.hor$Cdensity) * 1000
  #summary(al.hor$oc_d)
  al.horA = plyr::join(al.hor, al.gps, by=c("site"))
  al.col = c("profile", "description", "site_obsdate", "Longitude", "Latitude", "sampleID", "layer_sequence", "hzn_top", "depth", "Hcode", "tex_psda", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "wpg2", "BDfine", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = al.col[which(!al.col %in% names(al.horA))]
  if(length(x.na)>0){ for(i in x.na){ al.horA[,i] = NA } }
  chemsprops.Alaska = al.horA[,al.col]
  chemsprops.Alaska$source_db = "Alaska_interior"
  chemsprops.Alaska$confidence_degree = 1
  chemsprops.Alaska$project_url = "https://www.usgs.gov/centers/gmeg"
  chemsprops.Alaska$citation_url = "https://doi.org/10.5194/essd-12-1745-2020"
  chemsprops.Alaska = complete.vars(chemsprops.Alaska, sel = c("oc","oc_d"), coords = c("Longitude", "Latitude"))  
}
dim(chemsprops.Alaska)
#> [1] 3882   36
```

#### Croatian Soil Pedon data

- Martinović J., (2000) ["Tla u Hrvatskoj"](https://books.google.nl/books?id=k_a2MgAACAAJ), Monografija, Državna uprava za zaštitu prirode i okoliša, str. 269, Zagreb. ISBN: 9536793059
- Bašić F., (2014) ["The Soils of Croatia"](https://books.google.nl/books?id=VbJEAAAAQBAJ). World Soils Book Series, Springer Science & Business Media, 179 pp. ISBN: 9400758154


```r
if(!exists("chemsprops.bpht")){
  bpht.site <- read.csv("/mnt/diskstation/data/Soil_points/Croatia/WBSoilHR_sites_1997.csv", stringsAsFactors = FALSE)
  bpht.hors <- read.csv("/mnt/diskstation/data/Soil_points/Croatia/WBSoilHR_1997.csv", stringsAsFactors = FALSE)
  ## filter typos
  for(j in c("GOR", "DON", "MKP", "PH1", "PH2", "MSP", "MP", "MG", "HUM", "EXTN", "EXTP", "EXTK", "CAR")){
    bpht.hors[,j] = as.numeric(bpht.hors[,j])
  }
  ## Convert to the USDA standard
  bpht.hors$sand_tot_psa <- bpht.hors$MSP * 0.8 + bpht.hors$MKP
  bpht.hors$silt_tot_psa <- bpht.hors$MP + bpht.hors$MSP * 0.2
  bpht.hors$oc <- signif(bpht.hors$HUM/1.724 * 10, 3)
  ## summary(bpht.hors$sand_tot_psa)
  bpht.s.lst <- c("site_key", "UZORAK", "Cro16.30_X", "Cro16.30_Y", "FITOC", "STIJENA", "HID_DREN", "DUBINA")
  bpht.hor = plyr::join(bpht.site[,bpht.s.lst], bpht.hors)
  bpht.hor$wpg2 = bpht.hor$STIJENA
  bpht.hor$DON <- ifelse(is.na(bpht.hor$DON), bpht.hor$GOR+50, bpht.hor$DON)
  bpht.hor$depth <- bpht.hor$GOR + (bpht.hor$DON - bpht.hor$GOR)/2
  bpht.hor = bpht.hor[!is.na(bpht.hor$depth),]
  bpht.hor$wpg2[which(bpht.hor$GOR<30)] <- bpht.hor$wpg2[which(bpht.hor$GOR<30)]*.3
  bpht.hor$sample_key = make.unique(paste(bpht.hor$PEDOL_ID, bpht.hor$OZN, sep="_"))
  bpht.hor$sand_tot_psa[bpht.hor$sample_key=="805_Amo"] <- bpht.hor$sand_tot_psa[bpht.hor$sample_key=="805_Amo"]/10
  ## convert N, P, K
  #summary(bpht.hor$EXTK) -- measurements units?
  bpht.hor$p_ext = bpht.hor$EXTP * 4.364
  bpht.hor$k_ext = bpht.hor$EXTK * 8.3013
  bpht.hor = bpht.hor[!is.na(bpht.hor$`Cro16.30_X`),]
  ## coordinates:
  bpht.pnts = SpatialPointsDataFrame(bpht.hor[,c("Cro16.30_X","Cro16.30_Y")], bpht.hor["site_key"], proj4string = CRS("+proj=tmerc +lat_0=0 +lon_0=16.5 +k=0.9999 +x_0=2500000 +y_0=0 +ellps=bessel +towgs84=550.499,164.116,475.142,5.80967,2.07902,-11.62386,0.99999445824 +units=m"))
  bpht.pnts.ll <- spTransform(bpht.pnts, CRS("+proj=longlat +datum=WGS84"))
  bpht.hor$longitude_decimal_degrees = bpht.pnts.ll@coords[,1]
  bpht.hor$latitude_decimal_degrees = bpht.pnts.ll@coords[,2] 
  bpht.h.lst <- c('site_key', 'OZ_LIST_PROF', 'UZORAK', 'longitude_decimal_degrees', 'latitude_decimal_degrees', 'labsampnum', 'layer_sequence', 'GOR', 'DON', 'OZN', 'TT', 'MG', 'silt_tot_psa', 'sand_tot_psa', 'oc', 'oc_d', 'c_tot', 'EXTN', 'PH2', 'PH1', 'ph_cacl2', 'cec_sum', 'cec_nh4', 'ecec', 'wpg2', 'db_od', 'ca_ext', 'mg_ext', 'na_ext', 'k_ext', 'ec_satp', 'ec_12pre')
  x.na = bpht.h.lst[which(!bpht.h.lst %in% names(bpht.hor))]
  if(length(x.na)>0){ for(i in x.na){ bpht.hor[,i] = NA } }
  chemsprops.bpht = bpht.hor[,bpht.h.lst]
  chemsprops.bpht$source_db = "Croatian_Soil_Pedon"
  chemsprops.bpht$confidence_degree = 1
  chemsprops.bpht$project_url = "http://www.haop.hr/"
  chemsprops.bpht$citation_url = "https://books.google.nl/books?id=k_a2MgAACAAJ"
  chemsprops.bpht = complete.vars(chemsprops.bpht, sel = c("oc","MG","PH1","k_ext"))
}
dim(chemsprops.bpht)
#> [1] 5746   36
```


#### Remnant native SOC database

- Sanderman, J., (2017) "Remnant native SOC database for release.xlsx", Soil carbon profile data from paired land use comparisons, https://doi.org/10.7910/DVN/QQQM8V/8MSBNI, Harvard Dataverse, V1


```r
if(!exists("chemsprops.RemnantSOC")){
  rem.hor <- openxlsx::read.xlsx("/mnt/diskstation/data/Soil_points/INT/WHRC_remnant_SOC/remnant+native+SOC+database+for+release.xlsx", sheet = 3)
  rem.site <- openxlsx::read.xlsx("/mnt/diskstation/data/Soil_points/INT/WHRC_remnant_SOC/remnant+native+SOC+database+for+release.xlsx", sheet = 2)
  rem.ref <- openxlsx::read.xlsx("/mnt/diskstation/data/Soil_points/INT/WHRC_remnant_SOC/remnant+native+SOC+database+for+release.xlsx", sheet = 4)
  rem.site = plyr::join(rem.site, rem.ref[,c("Source.No.","DOI","Sample_year")], by=c("Source.No."))
  rem.site$Site = rem.site$Site.ID
  rem.horA = plyr::join(rem.hor, rem.site, by=c("Site"))
  rem.horA$hzn_top = rem.horA$'U_depth.(m)'*100
  rem.horA$hzn_bot = rem.horA$'L_depth.(m)'*100
  rem.horA$db_od = ifelse(is.na(as.numeric(rem.horA$'measured.BD.(Mg/m3)')), as.numeric(rem.horA$'estimated.BD.(Mg/m3)'), as.numeric(rem.horA$'measured.BD.(Mg/m3)'))
  rem.horA$oc_d = signif(rem.horA$'OC.(g/kg)' * rem.horA$db_od, 3)
  #summary(rem.horA$oc_d)
  rem.col = c("Source.No.", "Site", "Sample_year", "Longitude", "Latitude", "labsampnum", "layer_sequence", "hzn_top", "hzn_bot", "hzn_desgn", "tex_psda", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", "OC.(g/kg)", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "wpg2", "db_od", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = rem.col[which(!rem.col %in% names(rem.horA))]
  if(length(x.na)>0){ for(i in x.na){ rem.horA[,i] = NA } }
  chemsprops.RemnantSOC = rem.horA[,rem.col]
  chemsprops.RemnantSOC$source_db = "WHRC_remnant_SOC"
  chemsprops.RemnantSOC$confidence_degree = 8
  chemsprops.RemnantSOC$project_url = "https://www.woodwellclimate.org/research-area/carbon/"
  chemsprops.RemnantSOC$citation_url = "http://dx.doi.org/10.1073/pnas.1706103114"
  chemsprops.RemnantSOC = complete.vars(chemsprops.RemnantSOC, sel = c("OC.(g/kg)","oc_d"), coords = c("Longitude", "Latitude"))  
}
dim(chemsprops.RemnantSOC)
#> [1] 1604   36
```

#### Soil Health DB

- Jian, J., Du, X., & Stewart, R. D. (2020). A database for global soil health assessment. Scientific Data, 7(1), 1-8. https://doi.org/10.1038/s41597-020-0356-3. Data download URL: https://github.com/jinshijian/SoilHealthDB

Note: some information is available about column names (https://www.nature.com/articles/s41597-020-0356-3/tables/3) but detailed explanation is missing.


```r
if(!exists("chemsprops.SoilHealthDB")){
  shdb.hor <- openxlsx::read.xlsx("/mnt/diskstation/data/Soil_points/INT/SoilHealthDB/SoilHealthDB_V2.xlsx", sheet = 1, na.strings = c("NA", "NotAvailable", "Not-available"))
  #summary(as.factor(shdb.hor$SamplingDepth))
  shdb.hor$hzn_top = as.numeric(sapply(shdb.hor$SamplingDepth, function(i){ strsplit(i, "-to-")[[1]][1] }))
  shdb.hor$hzn_bot = as.numeric(sapply(shdb.hor$SamplingDepth, function(i){ strsplit(i, "-to-")[[1]][2] }))
  shdb.hor$hzn_top = ifelse(is.na(shdb.hor$hzn_top), 0, shdb.hor$hzn_top)
  shdb.hor$hzn_bot = ifelse(is.na(shdb.hor$hzn_bot), 15, shdb.hor$hzn_bot)
  shdb.hor$oc = as.numeric(shdb.hor$BackgroundSOC) * 10
  shdb.hor$oc_d = signif(shdb.hor$oc * shdb.hor$SoilBD, 3)
  for(j in c("ClayPerc", "SiltPerc", "SandPerc", "SoilpH")){   shdb.hor[,j]  <- as.numeric(shdb.hor[,j])   }
  #summary(shdb.hor$oc_d)
  shdb.col = c("StudyID", "ExperimentID", "SamplingYear", "Longitude", "Latitude", "labsampnum", "layer_sequence", "hzn_top", "hzn_bot", "hzn_desgn", "Texture", "ClayPerc", "SiltPerc", "SandPerc", "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "SoilpH", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "wpg2", "SoilBD", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = shdb.col[which(!shdb.col %in% names(shdb.hor))]
  if(length(x.na)>0){ for(i in x.na){ shdb.hor[,i] = NA } }
  chemsprops.SoilHealthDB = shdb.hor[,shdb.col]
  chemsprops.SoilHealthDB$source_db = "SoilHealthDB"
  chemsprops.SoilHealthDB$confidence_degree = 8
  chemsprops.SoilHealthDB$project_url = "https://github.com/jinshijian/SoilHealthDB"
  chemsprops.SoilHealthDB$citation_url = "https://doi.org/10.1038/s41597-020-0356-3"
  chemsprops.SoilHealthDB = complete.vars(chemsprops.SoilHealthDB, sel = c("ClayPerc", "SoilpH", "oc"), coords = c("Longitude", "Latitude"))  
}
dim(chemsprops.SoilHealthDB)
#> [1] 120  36
```

#### Global Harmonized Dataset of SOC change under perennial crops

- Ledo, A., Hillier, J., Smith, P. et al. (2019) A global, empirical, harmonised dataset of soil organic carbon changes under perennial crops. Sci Data 6, 57. https://doi.org/10.1038/s41597-019-0062-1. Data download URL: https://doi.org/10.6084/m9.figshare.7637210.v2

Note: Many missing years for PREVIOUS SOC AND SOIL CHARACTERISTICS.


```r
if(!exists("chemsprops.SOCPDB")){
  library("readxl")
  socpdb <- readxl::read_excel("/mnt/diskstation/data/Soil_points/INT/SOCPDB/SOC_perennials_DATABASE.xls", skip=1, sheet = 1)
  #names(socpdb)
  #summary(as.numeric(socpdb$year_measure))
  socpdb$year_measure = ifelse(is.na(as.numeric(socpdb$year_measure)), as.numeric(socpdb$yearPpub)-5, as.numeric(socpdb$year_measure))
  socpdb$year_measure = ifelse(socpdb$year_measure<1960, NA, socpdb$year_measure)
  socpdb$depth_current = socpdb$soil_to_cm_current - socpdb$soil_from_cm_current
  socpdb = socpdb[socpdb$depth_current>5,]
  socpdb$SOC_g_kg_current = ifelse(is.na(as.numeric(socpdb$SOC_g_kg_current)), as.numeric(socpdb$SOC_Mg_ha_current) / (socpdb$depth_current/100 * as.numeric(socpdb$bulk_density_Mg_m3_current) * 1000) * 10, as.numeric(socpdb$SOC_g_kg_current))
  socpdb$depth_previous = socpdb$soil_to_cm_previous - socpdb$soil_from_cm_previous
  socpdb$SOC_g_kg_previous = ifelse(is.na(as.numeric(socpdb$SOC_g_kg_previous)), as.numeric(socpdb$SOC_Mg_ha_previous) / (socpdb$depth_previous/100 * as.numeric(socpdb$Bulkdensity_previous) * 1000) * 10, as.numeric(socpdb$SOC_g_kg_previous))
  hor.b = which(names(socpdb) %in% c("ID", "plotID", "Longitud", "Latitud", "year_measure", "years_since_luc", "USDA", "original_source"))
  socpdb1 = socpdb[,c(hor.b, grep("_current", names(socpdb)))]
  #summary(as.numeric(socpdb1$years_since_luc))
  ## 10 yrs median
  socpdb1$site_obsdate = socpdb1$year_measure
  socpdb2 = socpdb[,c(hor.b, grep("_previous", names(socpdb)))]
  socpdb2$site_obsdate = socpdb2$year_measure - ifelse(is.na(as.numeric(socpdb2$years_since_luc)), 10, as.numeric(socpdb2$years_since_luc))
  colnames(socpdb2) <- sub("_previous", "_current", colnames(socpdb2))
  nm.socpdb = c("site_key", "usiteid", "site_obsdate", "longitude_decimal_degrees", "latitude_decimal_degrees", "hzn_top", "hzn_bot", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", "oc", "ph_h2o", "db_od")
  sel.socdpb1 = c("ID", "original_source", "site_obsdate", "Longitud", "Latitud", "soil_from_cm_current", "soil_to_cm_current", "%clay_current", "%silt_current", "%sand_current", "SOC_g_kg_current", "ph_current", "bulk_density_Mg_m3_current")
  sel.socdpb2 = c("ID", "original_source", "site_obsdate", "Longitud", "Latitud", "soil_from_cm_current", "soil_to_cm_current", "%clay_current", "%silt_current", "%sand_current", "SOC_g_kg_current", "ph_current", "Bulkdensity_current")   
  socpdbALL = as.data.frame(dplyr::bind_rows(lapply(list(socpdb1[,sel.socdpb1], socpdb2[,sel.socdpb2]), function(i){ dplyr::mutate_all(setNames(i, nm.socpdb), as.character) })))
  for(j in 1:ncol(socpdbALL)){ socpdbALL[,j] <- as.numeric(socpdbALL[,j]) }
  #summary(socpdbALL$oc) ## mean = 15
  #summary(socpdbALL$db_od)
  #summary(socpdbALL$ph_h2o)
  socpdbALL$oc_d = signif(socpdbALL$oc * socpdbALL$db_od, 3)
  #summary(socpdbALL$oc_d)
  x.na = col.names[which(!col.names %in% names(socpdbALL))]
  if(length(x.na)>0){ for(i in x.na){ socpdbALL[,i] = NA } }
  chemsprops.SOCPDB <- socpdbALL[,col.names]
  chemsprops.SOCPDB$source_db = "SOCPDB"
  chemsprops.SOCPDB$confidence_degree = 5
  chemsprops.SOCPDB$project_url = "https://africap.info/"
  chemsprops.SOCPDB$citation_url = "https://doi.org/10.1038/s41597-019-0062-1"
  chemsprops.SOCPDB = complete.vars(chemsprops.SOCPDB, sel = c("oc","ph_h2o","clay_tot_psa"))
}
dim(chemsprops.SOCPDB)
#> [1] 1526   36
```


#### Stocks of organic carbon in German agricultural soils (BZE_LW)

- Poeplau, C., Jacobs, A., Don, A., Vos, C., Schneider, F., Wittnebel, M., ... & Flessa, H. (2020). [Stocks of organic carbon in German agricultural soils—Key results of the first comprehensive inventory](https://doi.org/10.1002/jpln.202000113). Journal of Plant Nutrition and Soil Science, 183(6), 665-681. https://doi.org/10.1002/jpln.202000113. Data download URL: https://doi.org/10.3220/DATA20200203151139

Note: For protection of data privacy, the coordinate was randomly generated within a radius of 4-km around the planned sampling point. This data is hence probably not suitable for spatial analysis, predictive soil mapping.


```r
if(!exists("chemsprops.BZE_LW")){
  site.de <- openxlsx::read.xlsx("/mnt/diskstation/data/Soil_points/Germany/SITE.xlsx", sheet = 1)
  site.de$site_obsdate = format(as.Date(paste0("01-", site.de$Sampling_month, "-", site.de$Sampling_year), format="%d-%m-%Y"), "%Y-%m-%d")
  site.de.xy = site.de[,c("PointID","xcoord","ycoord")]
  ## 3104
  coordinates(site.de.xy) <- ~xcoord+ycoord
  proj4string(site.de.xy) <- CRS("+proj=utm +zone=32 +ellps=WGS84 +datum=WGS84 +units=m +no_defs")
  site.de.ll <- data.frame(spTransform(site.de.xy, CRS("+proj=longlat +ellps=WGS84 +datum=WGS84")))
  site.de$longitude_decimal_degrees = site.de.ll[,2]
  site.de$latitude_decimal_degrees = site.de.ll[,3]
  hor.de <- openxlsx::read.xlsx("/mnt/diskstation/data/Soil_points/Germany/LABORATORY_DATA.xlsx", sheet = 1)
  #hor.de = plyr::join(openxlsx::read.xlsx("/mnt/diskstation/data/Soil_points/Germany/LABORATORY_DATA.xlsx", sheet = 1), openxlsx::read.xlsx("/mnt/diskstation/data/Soil_points/Germany/HORIZON_DATA.xlsx", sheet = 1), by="PointID")
  ## 17,189 rows
  horALL.de = plyr::join(hor.de, site.de, by="PointID")
  ## Sand content [Mass-%]; grain size 63-2000µm (DIN ISO 11277)
  horALL.de$sand_tot_psa <- horALL.de$gS + horALL.de$mS + horALL.de$fS + 0.2 * horALL.de$gU
  horALL.de$silt_tot_psa <- horALL.de$fU + horALL.de$mU + 0.8 * horALL.de$gU
  ## Convert millisiemens/meter [mS/m] to microsiemens/centimeter [μS/cm, uS/cm]
  horALL.de$ec_satp = horALL.de$EC_H2O / 10
  hor.sel.de <- c("PointID", "Main.soil.type", "site_obsdate", "longitude_decimal_degrees", "latitude_decimal_degrees", "labsampnum", "layer_sequence", "Layer.upper.limit", "Layer.lower.limit", "hzn_desgn", "Soil.texture.class", "Clay", "silt_tot_psa", "sand_tot_psa", "TOC", "oc_d", "TC", "TN", "ph_kcl", "pH_H2O", "pH_CaCl2", "cec_sum", "cec_nh4", "ecec", "Rock.fragment.fraction", "BD_FS", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  #summary(horALL.de$TOC) ## mean = 12.3
  #summary(horALL.de$BD_FS) ## mean = 1.41
  #summary(horALL.de$pH_H2O)
  horALL.de$oc_d = signif(horALL.de$TOC * horALL.de$BD_FS * (1-horALL.de$Rock.fragment.fraction/100), 3)
  #summary(horALL.de$oc_d)
  x.na = hor.sel.de[which(!hor.sel.de %in% names(horALL.de))]
  if(length(x.na)>0){ for(i in x.na){ horALL.de[,i] = NA } }
  chemsprops.BZE_LW <- horALL.de[,hor.sel.de]
  chemsprops.BZE_LW$source_db = "BZE_LW"
  chemsprops.BZE_LW$confidence_degree = 3
  chemsprops.BZE_LW$project_url = "https://www.thuenen.de/de/ak/"
  chemsprops.BZE_LW$citation_url = "https://doi.org/10.1002/jpln.202000113"
  chemsprops.BZE_LW = complete.vars(chemsprops.BZE_LW, sel = c("TOC", "pH_H2O", "Clay"))
}
dim(chemsprops.BZE_LW)
#> [1] 17187    36
```


#### AARDEWERK-Vlaanderen-2010

- Beckers, V., Jacxsens, P., Van De Vreken, Ph., Van Meirvenne, M., Van Orshoven, J. (2011). Gebruik en installatie van de bodemdatabank AARDEWERK-Vlaanderen-2010. Spatial Applications Division Leuven, Belgium. Data download URL: https://www.dov.vlaanderen.be/geonetwork/home/api/records/78e15dd4-8070-4220-afac-258ea040fb30
- Ottoy, S., Beckers, V., Jacxsens, P., Hermy, M., & Van Orshoven, J. (2015). [Multi-level statistical soil profiles for assessing regional soil organic carbon stocks](https://doi.org/10.1016/j.geoderma.2015.04.001). Geoderma, 253, 12-20. https://doi.org/10.1016/j.geoderma.2015.04.001  


```r
if(!exists("chemsprops.Vlaanderen2010")){
  site.vl <- read.csv("/mnt/diskstation/data/Soil_points/Belgium/Vlaanderen/Aardewerk-Vlaanderen-2010_Profiel.csv")
  site.vl$site_obsdate = format(as.Date(sapply(site.vl$Profilering_Datum, function(i){strsplit(i, " ")[[1]][1]}), format="%d-%m-%Y"), "%Y-%m-%d")
  site.vl.xy = site.vl[,c("ID","Coordinaat_Lambert72_X","Coordinaat_Lambert72_Y")]
  ## 7020
  site.vl.xy = site.vl.xy[complete.cases(site.vl.xy),]
  coordinates(site.vl.xy) <- ~Coordinaat_Lambert72_X+Coordinaat_Lambert72_Y
  proj4string(site.vl.xy) <- CRS("+init=epsg:31300")
  site.vl.ll <- data.frame(spTransform(site.vl.xy, CRS("+proj=longlat +ellps=WGS84 +datum=WGS84")))
  site.vl$longitude_decimal_degrees = join(site.vl["ID"], site.vl.ll, by="ID")$Coordinaat_Lambert72_X
  site.vl$latitude_decimal_degrees = join(site.vl["ID"], site.vl.ll, by="ID")$Coordinaat_Lambert72_Y
  site.vl$Profiel_ID = site.vl$ID
  hor.vl <- read.csv("/mnt/diskstation/data/Soil_points/Belgium/Vlaanderen/Aardewerk-Vlaanderen-2010_Horizont.csv")
  ## 42,529 rows
  horALL.vl = plyr::join(hor.vl, site.vl, by="Profiel_ID")
  horALL.vl$oc = horALL.vl$Humus*10 /1.724
  summary(horALL.vl$oc) ## mean = 7.8
  #summary(horALL.vl$pH_H2O)
  horALL.vl$hzn_top <- rowSums(horALL.vl[,c("Diepte_grens_boven1", "Diepte_grens_boven2")], na.rm=TRUE)/2
  horALL.vl$hzn_bot <- rowSums(horALL.vl[,c("Diepte_grens_onder1","Diepte_grens_onder2")], na.rm=TRUE)/2
  horALL.vl$sand_tot_psa <- horALL.vl$T50_100 + horALL.vl$T100_200 + horALL.vl$T200_500 + horALL.vl$T500_1000 + horALL.vl$T1000_2000
  horALL.vl$silt_tot_psa <- horALL.vl$T2_10 + horALL.vl$T10_20 + horALL.vl$T20_50
  horALL.vl$tex_psda = paste0(horALL.vl$HorizontTextuur_code1, horALL.vl$HorizontTextuur_code2)
  ## some corrupt coordinates
  horALL.vl <- horALL.vl[horALL.vl$latitude_decimal_degrees > 50.6,]
  hor.sel.vl <- c("Profiel_ID", "Bodemgroep", "site_obsdate", "longitude_decimal_degrees", "latitude_decimal_degrees", "labsampnum", "Hor_nr", "hzn_top", "hzn_bot", "Naam", "tex_psda", "T0_2", "silt_tot_psa", "sand_tot_psa", "oc", "oc_d", "c_tot", "n_tot", "pH_KCl", "pH_H2O", "ph_cacl2", "Sorptiecapaciteit_Totaal", "cec_nh4", "ecec", "Tgroter_dan_2000", "db_od", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = hor.sel.vl[which(!hor.sel.vl %in% names(horALL.vl))]
  if(length(x.na)>0){ for(i in x.na){ horALL.vl[,i] = NA } }
  chemsprops.Vlaanderen <- horALL.vl[,hor.sel.vl]
  chemsprops.Vlaanderen$source_db = "Vlaanderen"
  chemsprops.Vlaanderen$confidence_degree = 2
  chemsprops.Vlaanderen$project_url = "https://www.dov.vlaanderen.be"
  chemsprops.Vlaanderen$citation_url = "https://doi.org/10.1016/j.geoderma.2015.04.001"
  chemsprops.Vlaanderen = complete.vars(chemsprops.Vlaanderen, sel = c("oc", "pH_H2O", "T0_2"))
}
dim(chemsprops.Vlaanderen)
#> [1] 41310    36
```


#### Chilean Soil Organic Carbon database

- Pfeiffer, M., Padarian, J., Osorio, R., Bustamante, N., Olmedo, G. F., Guevara, M., et al. (2020) [CHLSOC: the Chilean Soil Organic Carbon database, a multi-institutional collaborative effort](https://doi.org/10.5194/essd-12-457-2020). Earth Syst. Sci. Data, 12, 457–468, https://doi.org/10.5194/essd-12-457-2020. Data download URL: https://doi.org/10.17605/OSF.IO/NMYS3


```r
if(!exists("chemsprops.CHLSOC")){
  chl.hor <- read.csv("/mnt/diskstation/data/Soil_points/Chile/CHLSOC/CHLSOC_v1.0.csv", stringsAsFactors = FALSE)
  #summary(chl.hor$oc)
  chl.hor$oc = chl.hor$oc*10
  #summary(chl.hor$bd)
  chl.hor$oc_d = signif(chl.hor$oc  * chl.hor$bd * (100 - ifelse(is.na(chl.hor$crf), 0, chl.hor$crf))/100, 3)
  #summary(chl.hor$oc_d)
  chl.col = c("ProfileID", "usiteid", "year", "long", "lat", "labsampnum", "layer_sequence", "top", "bottom", "hzn_desgn", "tex_psda", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "ph_cacl2", "cec_sum", "cec_nh4", "ecec", "crf", "bd", "ca_ext", "mg_ext", "na_ext", "k_ext", "ec_satp", "ec_12pre")
  x.na = chl.col[which(!chl.col %in% names(chl.hor))]
  if(length(x.na)>0){ for(i in x.na){ chl.hor[,i] = NA } }
  chemsprops.CHLSOC = chl.hor[,chl.col]
  chemsprops.CHLSOC$source_db = "Chilean_SOCDB"
  chemsprops.CHLSOC$confidence_degree = 4
  chemsprops.CHLSOC$project_url = "https://doi.org/10.17605/OSF.IO/NMYS3"
  chemsprops.CHLSOC$citation_url = "https://doi.org/10.5194/essd-12-457-2020"
  chemsprops.CHLSOC = complete.vars(chemsprops.CHLSOC, sel = c("oc", "bd"), coords = c("long", "lat"))
}
dim(chemsprops.CHLSOC)
#> [1] 16371    36
```


#### Scotland (NSIS_1)

- Lilly, A., Bell, J.S., Hudson, G., Nolan, A.J. & Towers. W. (Compilers) (2010). National soil inventory of Scotland (NSIS_1); site location, sampling and profile description protocols. (1978-1988). Technical Bulletin. Macaulay Institute, Aberdeen. <https://doi.org/10.5281/zenodo.4650230>. Data download URL: <https://www.hutton.ac.uk/learning/natural-resource-datasets/soilshutton/soils-maps-scotland/download>


```r
if(!exists("chemsprops.ScotlandNSIS1")){
  sco.xy = read.csv("/mnt/diskstation/data/Soil_points/Scotland/NSIS_10km.csv")
  coordinates(sco.xy) = ~ easting + northing
  proj4string(sco.xy) = "EPSG:27700"
  sco.ll = as.data.frame(spTransform(sco.xy, CRS("EPSG:4326")))
  sco.ll$site_obsdate = as.numeric(sapply(sco.ll$profile_da, function(x){substr(x, nchar(x)-3, nchar(x))}))
  #hist(sco.ll$site_obsdate[sco.ll$site_obsdate>1000])
  ## no points after 1990!!
  #summary(sco.ll$exch_k)
  sco.in.name = c("profile_id", "site_obsdate", "easting", "northing", "horz_top", "horz_botto", 
                  "horz_symb", "sample_id", "texture_ps",
                  "sand_int", "silt_int", "clay", "carbon", "nitrogen", "ph_h2o", "exch_ca", 
                  "exch_mg", "exch_na", "exch_k", "sum_cation")
  #sco.in.name[which(!sco.in.name %in% names(sco.ll))]
  sco.x = as.data.frame(sco.ll[,sco.in.name])
  #sco.x = sco.x[!sco.x$sample_id==0,]
  #summary(sco.x$carbon)
  sco.out.name = c("usiteid", "site_obsdate", "longitude_decimal_degrees", "latitude_decimal_degrees",
                     "hzn_bot", "hzn_top", "hzn_desgn", "labsampnum", "tex_psda", "sand_tot_psa", "silt_tot_psa", 
                      "clay_tot_psa", "oc", "n_tot", "ph_h2o", "ca_ext",
                      "mg_ext", "na_ext", "k_ext", "cec_sum")
  ## translate values
  sco.fun.lst = as.list(rep("as.numeric(x)*1", length(sco.in.name)))
  sco.fun.lst[[which(sco.in.name=="profile_id")]] = "paste(x)"
  sco.fun.lst[[which(sco.in.name=="exch_ca")]] = "as.numeric(x)*200"
  sco.fun.lst[[which(sco.in.name=="exch_mg")]] = "as.numeric(x)*121"
  sco.fun.lst[[which(sco.in.name=="exch_k")]] = "as.numeric(x)*391"
  sco.fun.lst[[which(sco.in.name=="exch_na")]] = "as.numeric(x)*230"
  sco.fun.lst[[which(sco.in.name=="carbon")]] = "as.numeric(x)*10"
  sco.fun.lst[[which(sco.in.name=="nitrogen")]] = "as.numeric(x)*10"
  ## save translation rules:
  write.csv(data.frame(sco.in.name, sco.out.name, unlist(sco.fun.lst)), "scotland_soilab_transvalues.csv")
  sco.soil = transvalues(sco.x, sco.out.name, sco.in.name, sco.fun.lst)
  x.na = col.names[which(!col.names %in% names(sco.soil))]
  if(length(x.na)>0){ for(i in x.na){ sco.soil[,i] = NA } }
  chemsprops.ScotlandNSIS1 = sco.soil[,col.names]
  chemsprops.ScotlandNSIS1$source_db = "ScotlandNSIS1"
  chemsprops.ScotlandNSIS1$confidence_degree = 2
  chemsprops.ScotlandNSIS1$project_url = "http://soils.environment.gov.scot/"
  chemsprops.ScotlandNSIS1$citation_url = "https://doi.org/10.5281/zenodo.4650230"
  chemsprops.ScotlandNSIS1 = complete.vars(chemsprops.ScotlandNSIS1, sel = c("oc", "ph_h2o", "clay_tot_psa"))
}
dim(chemsprops.ScotlandNSIS1)
#> [1] 2977   36
```

#### Ecoforest map of Quebec, Canada 

- Duchesne, L., Ouimet, R., (2021). Digital mapping of soil texture in ecoforest polygons in Quebec, Canada. PeerJ 9:e11685 <https://doi.org/10.7717/peerj.11685>. Data download URL: <https://doi.org/10.7717/peerj.11685/supp-1>


```r
if(!exists("chemsprops.QuebecTEX")){
  que.xy = read.csv("/mnt/diskstation/data/Soil_points/Canada/Quebec/RawData.csv")
  #summary(as.factor(que.xy$Horizon))
  ## horizon depths were not measured - we assume 15-30 and 30-80
  que.xy$hzn_top = ifelse(que.xy$Horizon=="B", 15, 30)
  que.xy$hzn_bot = ifelse(que.xy$Horizon=="B", 30, 80)
  que.xy$site_key = que.xy$usiteid
  que.xy$latitude_decimal_degrees = que.xy$Latitude
  que.xy$longitude_decimal_degrees = que.xy$Longitude
  que.xy$hzn_desgn = que.xy$Horizon
  que.xy$sand_tot_psa = que.xy$PC_Sand
  que.xy$silt_tot_psa = que.xy$PC_Silt
  que.xy$clay_tot_psa = que.xy$PC_Clay
  x.na = col.names[which(!col.names %in% names(que.xy))]
  if(length(x.na)>0){ for(i in x.na){ que.xy[,i] = NA } }
  chemsprops.QuebecTEX = que.xy[,col.names]
  chemsprops.QuebecTEX$source_db = "QuebecTEX"
  chemsprops.QuebecTEX$confidence_degree = 4
  chemsprops.QuebecTEX$project_url = ""
  chemsprops.QuebecTEX$citation_url = "https://doi.org/10.7717/peerj.11685"
  chemsprops.QuebecTEX = complete.vars(chemsprops.QuebecTEX, sel = c("clay_tot_psa"))
}
dim(chemsprops.QuebecTEX)
#> [1] 26648    36
```

#### Pseudo-observations

- Pseudo-observations using simulated points (world deserts)  


```r
if(!exists("chemsprops.SIM")){
  ## 0 soil organic carbon + 98% sand content (deserts)
  load("deserts.pnt.rda")
  nut.sim <- as.data.frame(spTransform(deserts.pnt, CRS("+proj=longlat +datum=WGS84")))
  nut.sim[,1] <- NULL
  nut.sim <- plyr::rename(nut.sim, c("x"="longitude_decimal_degrees", "y"="latitude_decimal_degrees"))
  nr = nrow(nut.sim)
  nut.sim$site_key <- paste("Simulated", 1:nr, sep="_")
  ## insert zeros for all nutrients except for the once we are not sure:
  ## http://www.decodedscience.org/chemistry-sahara-sand-elements-dunes/45828
  sim.vars = c("oc", "oc_d", "c_tot", "n_tot", "ecec", "clay_tot_psa", "mg_ext", "k_ext")
  nut.sim[,sim.vars] <- 0
  nut.sim$silt_tot_psa = 2
  nut.sim$sand_tot_psa = 98
  nut.sim$hzn_top = 0
  nut.sim$hzn_bot = 30
  nut.sim$db_od = 1.55
  nut.sim2 = nut.sim
  nut.sim2$silt_tot_psa = 1
  nut.sim2$sand_tot_psa = 99
  nut.sim2$hzn_top = 30
  nut.sim2$hzn_bot = 60
  nut.sim2$db_od = 1.6
  nut.simA = rbind(nut.sim, nut.sim2)
  #str(nut.simA)
  nut.simA$source_db = "Simulated"
  nut.simA$confidence_degree = 10
  x.na = col.names[which(!col.names %in% names(nut.simA))]
  if(length(x.na)>0){ for(i in x.na){ nut.simA[,i] = NA } }
  chemsprops.SIM = nut.simA[,col.names]
  chemsprops.SIM$project_url = "https://gitlab.com/openlandmap/"
  chemsprops.SIM$citation_url = "https://gitlab.com/openlandmap/compiled-ess-point-data-sets/"
}
dim(chemsprops.SIM)
#> [1] 718  36
```

Other potential large soil profile DBs of interest:

- Shangguan, W., Dai, Y., Liu, B., Zhu, A., Duan, Q., Wu, L., ... & Chen, D. (2013). [A China data set of soil properties for land surface modeling](https://doi.org/10.1002/jame.20026). Journal of Advances in Modeling Earth Systems, 5(2), 212-224.
- Salković, E., Djurović, I., Knežević, M., Popović-Bugarin, V., & Topalović, A. (2018). Digitization and mapping of national legacy soil data of Montenegro. Soil and Water Research, 13(2), 83-89. https://doi.org/10.17221/81/2017-SWR

## ![alt text](./tex/R_logo.svg.png "Bind everything") Bind all datasets

#### Bind and clean-up


```r
ls(pattern=glob2rx("chemsprops.*"))
#>  [1] "chemsprops.AfSIS1"        "chemsprops.AfSPDB"       
#>  [3] "chemsprops.Alaska"        "chemsprops.bpht"         
#>  [5] "chemsprops.BZE_LW"        "chemsprops.CAPERM"       
#>  [7] "chemsprops.CHLSOC"        "chemsprops.CNSOT"        
#>  [9] "chemsprops.CostaRica"     "chemsprops.CUFS"         
#> [11] "chemsprops.EGRPR"         "chemsprops.FEBR"         
#> [13] "chemsprops.FIADB"         "chemsprops.FRED"         
#> [15] "chemsprops.GEMAS"         "chemsprops.GROOT"        
#> [17] "chemsprops.IRANSPDB"      "chemsprops.ISCND"        
#> [19] "chemsprops.LandPKS"       "chemsprops.LUCAS"        
#> [21] "chemsprops.LUCAS2"        "chemsprops.Mangroves"    
#> [23] "chemsprops.NAMSOTER"      "chemsprops.NatSoil"      
#> [25] "chemsprops.NCSCD"         "chemsprops.NCSS"         
#> [27] "chemsprops.NPDB"          "chemsprops.Peatlands"    
#> [29] "chemsprops.PRONASOLOS"    "chemsprops.QuebecTEX"    
#> [31] "chemsprops.RaCA"          "chemsprops.RemnantSOC"   
#> [33] "chemsprops.ScotlandNSIS1" "chemsprops.SIM"          
#> [35] "chemsprops.SISLAC"        "chemsprops.SOCPDB"       
#> [37] "chemsprops.SoDaH"         "chemsprops.SoilHealthDB" 
#> [39] "chemsprops.SRDB"          "chemsprops.USGS.NGS"     
#> [41] "chemsprops.Vlaanderen"    "chemsprops.WISE"
tot_sprops = dplyr::bind_rows(lapply(ls(pattern=glob2rx("chemsprops.*")), function(i){ mutate_all(setNames(get(i), col.names), as.character) }))
## convert to numeric:
for(j in c("longitude_decimal_degrees", "latitude_decimal_degrees", "layer_sequence", 
           "hzn_top", "hzn_bot", "clay_tot_psa", "silt_tot_psa", "sand_tot_psa", 
           "oc", "oc_d", "c_tot", "n_tot", "ph_kcl", "ph_h2o", "ph_cacl2", "cec_sum", 
           "cec_nh4", "ecec", "wpg2", "db_od", "ca_ext", "mg_ext", "na_ext", "k_ext", 
           "ec_satp", "ec_12pre")){
  tot_sprops[,j] = as.numeric(tot_sprops[,j])
}
#> Warning: NAs introduced by coercion

#> Warning: NAs introduced by coercion

#> Warning: NAs introduced by coercion

#> Warning: NAs introduced by coercion

#> Warning: NAs introduced by coercion
#head(tot_sprops)
```

Clean up typos and physically impossible values:


```r
tex.rm = rowSums(tot_sprops[,c("clay_tot_psa", "sand_tot_psa", "silt_tot_psa")])
summary(tex.rm)
#>     Min.  1st Qu.   Median     Mean  3rd Qu.     Max.     NA's 
#> -2997.00   100.00   100.00    99.39   100.00   500.00   274336
for(j in c("clay_tot_psa", "sand_tot_psa", "silt_tot_psa", "wpg2")){
  tot_sprops[,j] = ifelse(tot_sprops[,j]>100|tot_sprops[,j]<0, NA, tot_sprops[,j])
  tot_sprops[,j] = ifelse(tex.rm<99|is.na(tex.rm)|tex.rm>101, NA, tot_sprops[,j])
}
for(j in c("ph_h2o","ph_kcl","ph_cacl2")){
  tot_sprops[,j] = ifelse(tot_sprops[,j]>12|tot_sprops[,j]<2, NA, tot_sprops[,j])
}
#hist(tot_sprops$db_od)
for(j in c("db_od")){
  tot_sprops[,j] = ifelse(tot_sprops[,j]>2.4|tot_sprops[,j]<0.05, NA, tot_sprops[,j])
}
#hist(tot_sprops$oc)
for(j in c("oc")){
  tot_sprops[,j] = ifelse(tot_sprops[,j]>800|tot_sprops[,j]<0, NA, tot_sprops[,j])
}
```

Fill-in the missing depths:


```r
## soil layer depth (middle)
tot_sprops$hzn_depth = tot_sprops$hzn_top + (tot_sprops$hzn_bot-tot_sprops$hzn_top)/2
summary(is.na(tot_sprops$hzn_depth))
#>    Mode   FALSE    TRUE 
#> logical  766689    5465
## Note: large number of horizons without a depth
tot_sprops = tot_sprops[!is.na(tot_sprops$hzn_depth),]
#quantile(tot_sprops$hzn_depth, c(0.01,0.99), na.rm=TRUE)
tot_sprops$hzn_depth = ifelse(tot_sprops$hzn_depth<0, 10, ifelse(tot_sprops$hzn_depth>800, 800, tot_sprops$hzn_depth))
#hist(tot_sprops$hzn_depth, breaks=45)
```

Summary number of points per data source:


```r
summary(as.factor(tot_sprops$source_db))
#>              AfSIS1              AfSPDB     Alaska_interior              BZE_LW 
#>                4162               60277                3880               17187 
#>         Canada_CUFS         Canada_NPDB    Canada_subarctic       Chilean_SOCDB 
#>               15162               14900                1180               16358 
#>         China_SOTER               CIFOR           CostaRica Croatian_Soil_Pedon 
#>                5105                 561                2029                5746 
#>       CSIRO_NatSoil                FEBR               FIADB                FRED 
#>               70688                7804               23208                 625 
#>          GEMAS_2009               GROOT           Iran_SPDB               ISCND 
#>                4131                 718                4677                3977 
#>          ISRIC_WISE             LandPKS          LUCAS_2009          LUCAS_2015 
#>               23278               41644               21272               21859 
#>         MangrovesDB            NAMSOTER               NCSCD          PRONASOLOS 
#>                7733                2941                7082               31655 
#>           QuebecTEX            RaCA2016        Russia_EGRPR       ScotlandNSIS1 
#>               26648               53663                4437                2977 
#>           Simulated              SISLAC              SOCPDB               SoDaH 
#>                 718               49416                1526               17766 
#>        SoilHealthDB                SRDB           USDA_NCSS            USGS.NGS 
#>                 120                1596              135671                9398 
#>          Vlaanderen    WHRC_remnant_SOC 
#>               41310                1604
```

Add unique row identifier


```r
tot_sprops$uuid = openssl::md5(make.unique(paste("OpenLandMap", tot_sprops$site_key, tot_sprops$layer_sequence, sep="_")))
```

and unique location based on the [Open Location Code](https://cran.r-project.org/web/packages/olctools/vignettes/Introduction_to_olctools.html):


```r
tot_sprops$olc_id = olctools::encode_olc(tot_sprops$latitude_decimal_degrees, tot_sprops$longitude_decimal_degrees, 11)
length(levels(as.factor(tot_sprops$olc_id)))
#> [1] 205687
## 205,620
```


```r
tot_sprops.pnts = tot_sprops[!duplicated(tot_sprops$olc_id),c("site_key", "source_db", "longitude_decimal_degrees", "latitude_decimal_degrees", "site_obsdate", "olc_id", "project_url", "citation_url")]
coordinates(tot_sprops.pnts) <- ~ longitude_decimal_degrees + latitude_decimal_degrees
proj4string(tot_sprops.pnts) <- "EPSG:4326"
```

Remove points falling in the sea or similar:


```r
if(!exists("ov.sprops")){
  #mask = terra::rast("./layers1km/lcv_landmask_esacci.lc.l4_c_1km_s0..0cm_2000..2015_v1.0.tif")
  mask = terra::rast("/mnt/diskstation/data/LandGIS/layers250m/lcv_landmask_esacci.lc.l4_c_250m_s0..0cm_2000..2015_v1.0.tif")
  ov.sprops <- terra::extract(mask, terra::vect(tot_sprops.pnts)) ## TAKES 2 mins
  summary(as.factor(ov.sprops[,2]))
  if(sum(is.na(ov.sprops[,2]))>0 | sum(ov.sprops[,2]==2)>0){
    rem.lst = which(is.na(ov.sprops[,2]) | ov.sprops[,2]==2 | ov.sprops[,2]==4)
    rem.sp = tot_sprops.pnts$site_key[rem.lst]
    tot_sprops.pnts = tot_sprops.pnts[-rem.lst,]
  }
}
## final number of unique spatial locations:
nrow(tot_sprops.pnts)
#> [1] 205687
## 203,107
```

#### Histogram plots

Have in mind that some datasets only represent top-soil (e.g. LUCAS) while other 
cover the whole soil depth, hence higher mean values for some regions (Europe) should 
be considered within the context of diverse soil depths.


```r
library(ggplot2)
ggplot(tot_sprops, aes(x=source_db, y=log1p(oc))) + geom_boxplot() + theme(axis.text.x = element_text(angle = 90, hjust = 1))
#> Warning: Removed 195328 rows containing non-finite values (stat_boxplot).
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-54-1.png" width="70%" style="display: block; margin: auto;" />


```r
sel.db = c("ISRIC_WISE", "Canada_CUFS", "USDA_NCSS", "AfSPDB", "Canada_NPDB", "FIADB", "PRONASOLOS", "CSIRO_NatSoil")
openair::scatterPlot(tot_sprops[tot_sprops$source_db %in% sel.db,], x = "hzn_depth", y = "oc", method = "hexbin", 
                     col = "increment", type = "source_db", log.x = TRUE, log.y=TRUE, ylab="SOC wprm", xlab="depth in cm")
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-55-1.png" width="70%" style="display: block; margin: auto;" />

Note: FIADB includes litter and the soil samples are taken at fixed depths. Canada 
dataset also shows SOC content for peatlands.


```r
ggplot(tot_sprops, aes(x=source_db, y=db_od)) + geom_boxplot() + theme(axis.text.x = element_text(angle = 90, hjust = 1))
#> Warning: Removed 537198 rows containing non-finite values (stat_boxplot).
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-56-1.png" width="70%" style="display: block; margin: auto;" />


```r
openair::scatterPlot(tot_sprops[tot_sprops$source_db %in% sel.db,], x = "oc", y = "db_od", method = "hexbin", 
                     col = "increment", type = "source_db", log.x=TRUE, ylab="Bulk density", xlab="SOC wprm")
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-57-1.png" width="70%" style="display: block; margin: auto;" />


```r
ggplot(tot_sprops, aes(x=source_db, y=log1p(oc_d))) + geom_boxplot() + theme(axis.text.x = element_text(angle = 90, hjust = 1))
#> Warning in log1p(oc_d): NaNs produced

#> Warning in log1p(oc_d): NaNs produced
#> Warning: Removed 561232 rows containing non-finite values (stat_boxplot).
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-58-1.png" width="70%" style="display: block; margin: auto;" />


```r
sel.db0 = c("ISRIC_WISE", "Canada_CUFS", "USDA_NCSS", "AfSPDB", "PRONASOLOS", "CSIRO_NatSoil")
openair::scatterPlot(tot_sprops[tot_sprops$source_db %in% sel.db0,], x = "oc", y = "oc_d", method = "hexbin", 
                     col = "increment", type = "source_db", log.x=TRUE, log.y=TRUE, xlab="SOC wprm", ylab="SOC kg/m3")
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-59-1.png" width="70%" style="display: block; margin: auto;" />

Note: SOC (%) and SOC density (kg/m3) are almost linear relationship (curving toward fixed value), 
except for organic soils (especially litter) where relationship is slightly shifted. 


```r
openair::scatterPlot(tot_sprops[tot_sprops$source_db %in% sel.db0,], x = "oc", y = "n_tot", method = "hexbin", 
                     col = "increment", type = "source_db", log.x=TRUE, log.y=TRUE, xlab="SOC wprm", ylab="N total wprm")
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-60-1.png" width="70%" style="display: block; margin: auto;" />


```r
ggplot(tot_sprops, aes(x=source_db, y=ph_h2o)) + geom_boxplot() + theme(axis.text.x = element_text(angle = 90, hjust = 1))
#> Warning: Removed 270562 rows containing non-finite values (stat_boxplot).
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-61-1.png" width="70%" style="display: block; margin: auto;" />


```r
openair::scatterPlot(tot_sprops[tot_sprops$source_db %in% c("ISRIC_WISE", "USDA_NCSS", "AfSPDB"),], x = "ph_kcl", y = "ph_h2o", method = "hexbin", 
                     col = "increment", type = "source_db", xlab="soil pH KCl", ylab="soil pH H2O")
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-62-1.png" width="70%" style="display: block; margin: auto;" />


```r
openair::scatterPlot(tot_sprops[tot_sprops$source_db %in% sel.db0,], x = "hzn_depth", y = "ph_h2o", method = "hexbin", 
                     col = "increment", type = "source_db", log.x = TRUE, log.y=TRUE, ylab="soil pH H2O", xlab="depth in cm")
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-63-1.png" width="70%" style="display: block; margin: auto;" />

Note: there seems to be no apparent correlation between soil pH and soil depth.


```r
openair::scatterPlot(tot_sprops[tot_sprops$source_db %in% c("ISRIC_WISE", "USDA_NCSS", "AfSPDB", "PRONASOLOS"),], x = "ph_h2o", y = "cec_sum", method = "hexbin", 
                     col = "increment", type = "source_db", log.y=TRUE, ylab="CEC", xlab="soil pH H2O")
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-64-1.png" width="70%" style="display: block; margin: auto;" />


```r
openair::scatterPlot(tot_sprops[tot_sprops$source_db %in% sel.db0,], x = "ph_h2o", y = "oc", method = "hexbin",  
                     col = "increment", type = "source_db", log.y=TRUE, ylab="SOC wprm", xlab="soil pH H2O")
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-65-1.png" width="70%" style="display: block; margin: auto;" />


```r
ggplot(tot_sprops, aes(x=source_db, y=clay_tot_psa)) + geom_boxplot() + theme(axis.text.x = element_text(angle = 90, hjust = 1))
#> Warning: Removed 279635 rows containing non-finite values (stat_boxplot).
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-66-1.png" width="70%" style="display: block; margin: auto;" />


```r
openair::scatterPlot(tot_sprops[tot_sprops$source_db %in% sel.db0,], y = "clay_tot_psa", x = "sand_tot_psa", method = "hexbin",  
                     col = "increment", type = "source_db", ylab="clay %", xlab="sand %")
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-67-1.png" width="70%" style="display: block; margin: auto;" />


```r
ggplot(tot_sprops, aes(x=source_db, y=log1p(cec_sum))) + geom_boxplot() + theme(axis.text.x = element_text(angle = 90, hjust = 1))
#> Warning in log1p(cec_sum): NaNs produced

#> Warning in log1p(cec_sum): NaNs produced
#> Warning: Removed 514240 rows containing non-finite values (stat_boxplot).
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-68-1.png" width="70%" style="display: block; margin: auto;" />


```r
ggplot(tot_sprops, aes(x=source_db, y=log1p(n_tot))) + geom_boxplot() + theme(axis.text.x = element_text(angle = 90, hjust = 1))
#> Warning in log1p(n_tot): NaNs produced

#> Warning in log1p(n_tot): NaNs produced
#> Warning: Removed 446028 rows containing non-finite values (stat_boxplot).
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-69-1.png" width="70%" style="display: block; margin: auto;" />


```r
ggplot(tot_sprops, aes(x=source_db, y=log1p(k_ext))) + geom_boxplot() + theme(axis.text.x = element_text(angle = 90, hjust = 1))
#> Warning in log1p(k_ext): NaNs produced

#> Warning in log1p(k_ext): NaNs produced
#> Warning: Removed 523010 rows containing non-finite values (stat_boxplot).
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-70-1.png" width="70%" style="display: block; margin: auto;" />


```r
ggplot(tot_sprops, aes(x=source_db, y=log1p(ec_satp))) + geom_boxplot() + theme(axis.text.x = element_text(angle = 90, hjust = 1))
#> Warning: Removed 608509 rows containing non-finite values (stat_boxplot).
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-71-1.png" width="70%" style="display: block; margin: auto;" />


```r
sprops_yrs = tot_sprops[!is.na(tot_sprops$site_obsdate),c("site_obsdate", "source_db", "oc")]
sprops_yrs$year = as.numeric(substr(x=sprops_yrs$site_obsdate, 1, 4))
#> Warning: NAs introduced by coercion
sprops_yrs$year = ifelse(sprops_yrs$year <1960, NA, ifelse(sprops_yrs$year>2024, NA, sprops_yrs$year))
```


```r
ggplot(sprops_yrs, aes(x=source_db, y=year)) + geom_boxplot() + theme(axis.text.x = element_text(angle = 90, hjust = 1))
#> Warning: Removed 70644 rows containing non-finite values (stat_boxplot).
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-73-1.png" width="70%" style="display: block; margin: auto;" />


```r
openair::scatterPlot(sprops_yrs[sprops_yrs$source_db %in% sel.db0,], y = "oc", x = "year", method = "hexbin",  
                     col = "increment", type = "source_db", log.y=TRUE, ylab="SOC wprm", xlab="Sampling year")
```

<img src="025-import_chemical_data_files/figure-html/unnamed-chunk-74-1.png" width="70%" style="display: block; margin: auto;" />

#### Convert to wide format

Add `layer_sequence` where missing since this is needed to be able to convert to wide 
format:


```r
#summary(tot_sprops$layer_sequence)
tot_sprops$dsiteid = paste(tot_sprops$source_db, tot_sprops$site_key, tot_sprops$site_obsdate, sep="_")
if(!exists("l.s1")){
  library(dplyr)
  ## Note: takes >2 mins
  l.s1 <- tot_sprops[,c("dsiteid","hzn_depth")] %>% group_by(dsiteid) %>% mutate(layer_sequence.f = data.table::frank(hzn_depth, ties.method = "first"))
  tot_sprops$layer_sequence.f = ifelse(is.na(tot_sprops$layer_sequence), l.s1$layer_sequence.f, tot_sprops$layer_sequence)
  tot_sprops$layer_sequence.f = ifelse(tot_sprops$layer_sequence.f>6, 6, tot_sprops$layer_sequence.f)
}
```

Convert long table to [wide table format](https://ncss-tech.github.io/AQP/aqp/aqp-intro.html) so that each depth gets unique column 
(note: the most computational / time-consuming step usually):


```r
if(!exists("tot_sprops.w")){
  library(data.table)
  tot_sprops.w = data.table::dcast( as.data.table(tot_sprops), 
                formula = olc_id ~ layer_sequence.f, 
                value.var = c("uuid", hor.names[-which(hor.names %in% c("site_key", "layer_sequence"))]), 
                ## "labsampnum", "hzn_desgn", "tex_psda" 
                #fun=function(x){ mean(x, na.rm=TRUE) }, 
                ## Note: does not work for characters
                fun=function(x){ x[1] },
                verbose = FALSE)
  ## remove "0" layers added automatically but containing no values
  tot_sprops.w = tot_sprops.w[,grep("*_0$", colnames(tot_sprops.w)):=NULL]
}
tot_sprops_w.pnts = tot_sprops.pnts
tot_sprops_w.pnts@data = plyr::join(tot_sprops.pnts@data, tot_sprops.w)
#> Joining by: olc_id
```

Write all soil profiles using a wide format:


```r
sel.rm.pnts  <- tot_sprops_w.pnts$source_db=="LUCAS_2009" | tot_sprops_w.pnts$source_db=="LUCAS_2015" | tot_sprops_w.pnts$site_key %in% mng.rm | tot_sprops_w.pnts$site_key %in% rem.sp
out.gpkg = "./out/gpkg/sol_chem.pnts_horizons.gpkg"
#unlink(out.gpkg)
if(!file.exists(out.gpkg)){
  writeOGR(tot_sprops_w.pnts[!sel.rm.pnts,], "./out/gpkg/sol_chem.pnts_horizons.gpkg", "sol_chem.pnts_horizons", drive="GPKG")
}
```


#### Save RDS files

Remove points that are not allowed to be distributed publicly:


```r
sel.rm  <- tot_sprops$source_db=="LUCAS_2009" | tot_sprops$source_db=="LUCAS_2015" | tot_sprops$site_key %in% mng.rm | tot_sprops$site_key %in% rem.sp
tot_sprops.s = tot_sprops[!sel.rm,] 
```

Plot in Goode Homolozine projection and save final objects:


```r
if(!file.exists("./img/sol_chem.pnts_sites.png")){
  tot_sprops.pnts_sf <- st_as_sf(tot_sprops.pnts[1], crs=4326)
  plot_gh(tot_sprops.pnts_sf, out.pdf="./img/sol_chem.pnts_sites.pdf")
  ## extremely slow --- takes 15mins
  system("pdftoppm ./img/sol_chem.pnts_sites.pdf ./img/sol_chem.pnts_sites -png -f 1 -singlefile")
  system("convert -crop 1280x575+36+114 ./img/sol_chem.pnts_sites.png ./img/sol_chem.pnts_sites.png")
}
```

<div class="figure" style="text-align: center">
<img src="./img/sol_chem.pnts_sites.png" alt="Soil profiles and soil samples with chemical and physical properties global compilation." width="100%" />
<p class="caption">(\#fig:sol_chem.pnts_sites)Soil profiles and soil samples with chemical and physical properties global compilation.</p>
</div>
Fig.  1: Soil profiles and soil samples with chemical and physical properties global compilation.

## Save final analysis-ready objects:


```r
saveRDS.gz(tot_sprops.s, "./out/rds/sol_chem.pnts_horizons.rds")
saveRDS.gz(tot_sprops, "/mnt/diskstation/data/Soil_points/sol_chem.pnts_horizons.rds")
saveRDS.gz(tot_sprops.pnts, "/mnt/diskstation/data/Soil_points/sol_chem.pnts_sites.rds")
#library(farff)
#writeARFF(tot_sprops.s, "./out/arff/sol_chem.pnts_horizons.arff", overwrite = TRUE)
## compressed CSV
write.csv(tot_sprops.s, file=gzfile("./out/csv/sol_chem.pnts_horizons.csv.gz"))
## regression matrix:
#saveRDS.gz(rm.sol, "./out/rds/sol_chem.pnts_horizons_rm.rds")
```

Save temp object:


```r
save.image.pigz(file="soilchem.RData")
## rmarkdown::render("Index.rmd")
```
