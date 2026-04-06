-- ============================================================================
-- Google Ads Campaign Data — ETL & Transformation Queries
-- Brave ISA: DA & DE Program | FY23–FY24
-- ============================================================================
-- Source: Google Ads exports (CSV → staging tables)
-- Target: Clean, unified analytics tables for BI reporting
-- Author: [Your Name]
-- ============================================================================


-- ============================================================================
-- STEP 1: CREATE STAGING TABLES (Raw Google Ads Exports)
-- ============================================================================

-- Keyword-level export (707 rows, primary dataset)
CREATE TABLE stg_keyword_report (
    campaign            VARCHAR(255),
    keyword_status      VARCHAR(50),
    keyword             VARCHAR(500),
    match_type          VARCHAR(50),
    ad_group            VARCHAR(100),
    status              VARCHAR(50),
    status_reasons      VARCHAR(500),
    conversions         VARCHAR(20),    -- loaded as string; may contain non-numeric
    currency_code       VARCHAR(10),
    cost_per_conv       DECIMAL(10,2),
    clicks              INT,
    cost                DECIMAL(10,2),
    impressions         INT,
    ctr                 VARCHAR(20),    -- contains ' --' for zero-impression rows
    avg_cpc             DECIMAL(10,2),
    quality_score       VARCHAR(10),    -- contains ' --' for most rows
    conv_rate           DECIMAL(10,4)
);

-- Campaign-level network/device breakdown (Raw sheet, left table)
CREATE TABLE stg_campaign_network (
    network             VARCHAR(100),
    click_type          VARCHAR(50),
    device              VARCHAR(50),
    campaign_status     VARCHAR(50),
    campaign            VARCHAR(255),
    budget              DECIMAL(10,2),
    campaign_type       VARCHAR(50),
    clicks              INT,
    impressions         INT,
    ctr                 DECIMAL(10,4),
    currency_code       VARCHAR(10),
    cost                DECIMAL(10,2),
    conversions         DECIMAL(10,2),
    cost_per_conv       DECIMAL(10,2),
    conv_rate           DECIMAL(10,4),
    avg_cpc             DECIMAL(10,4)
);

-- Ad group-level breakdown (Raw sheet, right table)
CREATE TABLE stg_adgroup_report (
    campaign            VARCHAR(255),
    adgroup_status      VARCHAR(50),
    ad_group            VARCHAR(100),
    status              VARCHAR(50),
    conversions         DECIMAL(10,2),
    cost_per_conv       DECIMAL(10,2),
    adgroup_type        VARCHAR(50),
    clicks              INT,
    impressions         INT,
    ctr                 DECIMAL(10,4),
    avg_cpc             DECIMAL(10,2),
    cost                DECIMAL(10,2),
    conv_rate           DECIMAL(10,4)
);

-- Demographic exports (3 separate pivot tables)
CREATE TABLE stg_demo_age (
    age_group           VARCHAR(20),
    campaign            VARCHAR(255),
    clicks              INT,
    impressions         INT,
    conversions         DECIMAL(10,2),
    cost                DECIMAL(10,2),
    true_cpa            DECIMAL(10,2),
    true_cvr            DECIMAL(10,4)
);

CREATE TABLE stg_demo_gender (
    gender              VARCHAR(20),
    campaign            VARCHAR(255),
    clicks              INT,
    impressions         INT,
    cost_per_conv       DECIMAL(10,2),
    conversions         DECIMAL(10,2),
    true_cpa            DECIMAL(10,2),
    true_cvr            DECIMAL(10,4)
);

CREATE TABLE stg_demo_income (
    income_bracket      VARCHAR(30),
    campaign            VARCHAR(255),
    conversions         DECIMAL(10,2),
    clicks              INT,
    impressions         INT,
    cost                DECIMAL(10,2),
    true_cpa            DECIMAL(10,2),
    true_cvr            DECIMAL(10,4)
);


-- ============================================================================
-- STEP 2: DATA CLEANING
-- ============================================================================
-- Issues found in raw data:
--   1. Campaign name has trailing whitespace: 'Trial 836 ' (extra space)
--   2. CTR column contains ' --' string for zero-impression keywords (521 rows)
--   3. Quality Score contains ' --' for 655 of 707 rows
--   4. 521 keywords have 0 impressions (never served)
--   5. Keywords use match type notation: [exact], "phrase", broad
--   6. Status reasons are semicolon-delimited multi-value strings
--   7. Conversions stored as string in some exports (fractional values like 7.67)

-- 2a. Clean keyword report — fix types, nulls, and whitespace
CREATE TABLE clean_keyword_report AS
SELECT
    -- Fix trailing whitespace in campaign names
    TRIM(campaign)                                          AS campaign,

    -- Extract a short campaign label for readability
    CASE
        WHEN TRIM(campaign) LIKE '%Trial 836%'  THEN 'FY23_DA_Trial'
        WHEN TRIM(campaign) LIKE '%DA Program%' THEN 'FY23_DA_Main'
        WHEN TRIM(campaign) LIKE '%DE Program%' THEN 'FY24_DE_Main'
    END                                                     AS campaign_label,

    -- Extract fiscal year
    CASE
        WHEN TRIM(campaign) LIKE 'FY23%' THEN 'FY23'
        WHEN TRIM(campaign) LIKE 'FY24%' THEN 'FY24'
    END                                                     AS fiscal_year,

    -- Extract program type
    CASE
        WHEN TRIM(campaign) LIKE '%DA Program%' OR
             TRIM(campaign) LIKE '%DA Program%Trial%' THEN 'Data Analytics'
        WHEN TRIM(campaign) LIKE '%DE Program%'        THEN 'Data Engineering'
    END                                                     AS program_type,

    keyword_status,

    -- Strip match type notation from keyword text for grouping
    CASE
        WHEN keyword LIKE '[%]'   THEN TRIM(BOTH ']' FROM TRIM(BOTH '[' FROM keyword))
        WHEN keyword LIKE '"%"'   THEN TRIM(BOTH '"' FROM keyword)
        ELSE TRIM(keyword)
    END                                                     AS keyword_clean,

    -- Keep original keyword with notation for match type reference
    TRIM(keyword)                                           AS keyword_original,

    match_type,
    TRIM(ad_group)                                          AS ad_group,
    status,

    -- Parse semicolon-delimited status reasons into clean flags
    CASE WHEN status_reasons LIKE '%rarely served%'     THEN 1 ELSE 0 END AS flag_rarely_served,
    CASE WHEN status_reasons LIKE '%low quality%'       THEN 1 ELSE 0 END AS flag_low_quality,
    CASE WHEN status_reasons LIKE '%ad group paused%'   THEN 1 ELSE 0 END AS flag_adgroup_paused,
    CASE WHEN status_reasons LIKE '%campaign paused%'   THEN 1 ELSE 0 END AS flag_campaign_paused,

    -- Cast conversions safely (some exports have fractional values)
    CAST(NULLIF(conversions, '') AS DECIMAL(10,2))          AS conversions,

    currency_code,
    clicks,
    cost,
    impressions,

    -- Replace ' --' sentinel with NULL for numeric fields
    CASE
        WHEN ctr = ' --' OR impressions = 0 THEN NULL
        ELSE CAST(ctr AS DECIMAL(10,4))
    END                                                     AS ctr,

    avg_cpc,

    CASE
        WHEN quality_score = ' --' THEN NULL
        ELSE CAST(quality_score AS INT)
    END                                                     AS quality_score,

    conv_rate

FROM stg_keyword_report
WHERE campaign IS NOT NULL;


-- 2b. Clean demographic tables — standardize names
CREATE TABLE clean_demo_age AS
SELECT
    age_group,
    TRIM(campaign)                                          AS campaign,
    CASE
        WHEN TRIM(campaign) LIKE '%Trial 836%'  THEN 'FY23_DA_Trial'
        WHEN TRIM(campaign) LIKE '%DA Program%' THEN 'FY23_DA_Main'
        WHEN TRIM(campaign) LIKE '%DE Program%' THEN 'FY24_DE_Main'
    END                                                     AS campaign_label,
    clicks,
    impressions,
    conversions,
    cost,
    true_cpa,
    true_cvr
FROM stg_demo_age
WHERE age_group NOT IN ('Total')
  AND campaign IS NOT NULL;


-- ============================================================================
-- STEP 3: COMPUTED METRICS & BUSINESS LOGIC
-- ============================================================================
-- Custom calculated fields match the pivot table formulas documented in Sheet7:
--   CPA  = IF(Conversions=0, IF(Cost>0, "WASTE", 0), Cost/Conversions)
--   CVR  = IF(Conversions=0, IF(Clicks=0, "WASTE", 0), Conversions/Clicks)

-- 3a. Keyword-level analytics table with calculated metrics
CREATE TABLE analytics_keyword AS
SELECT
    campaign_label,
    fiscal_year,
    program_type,
    keyword_clean,
    keyword_original,
    match_type,
    ad_group,
    keyword_status,
    status,
    impressions,
    clicks,
    conversions,
    cost,
    avg_cpc,
    quality_score,

    -- Calculated CTR (handle division by zero)
    CASE
        WHEN impressions > 0 THEN ROUND(clicks * 1.0 / impressions, 4)
        ELSE NULL
    END                                                     AS calc_ctr,

    -- Calculated CVR with WASTE flag
    CASE
        WHEN conversions > 0           THEN ROUND(conversions * 1.0 / clicks, 4)
        WHEN conversions = 0 AND clicks > 0 THEN 0     -- spent but no conversion
        ELSE NULL                                       -- no activity
    END                                                     AS calc_cvr,

    -- Calculated CPA with WASTE flag
    CASE
        WHEN conversions > 0           THEN ROUND(cost / conversions, 2)
        WHEN conversions = 0 AND cost > 0   THEN -1     -- WASTE indicator
        ELSE NULL
    END                                                     AS calc_cpa,

    -- Flag wasted spend (clicks or cost but zero conversions)
    CASE
        WHEN conversions = 0 AND cost > 0 THEN 1
        ELSE 0
    END                                                     AS is_wasted_spend,

    -- Performance tier for quick filtering
    CASE
        WHEN conversions > 0 AND (cost / NULLIF(conversions, 0)) <= 35 THEN 'High'
        WHEN conversions > 0 AND (cost / NULLIF(conversions, 0)) <= 55 THEN 'Medium'
        WHEN conversions > 0 THEN 'Low'
        WHEN cost > 0        THEN 'Waste'
        ELSE 'Inactive'
    END                                                     AS performance_tier

FROM clean_keyword_report;


-- ============================================================================
-- STEP 4: AGGREGATED VIEWS FOR BI REPORTING
-- ============================================================================

-- 4a. Campaign-level summary
CREATE VIEW vw_campaign_summary AS
SELECT
    campaign_label,
    fiscal_year,
    program_type,
    SUM(impressions)                                        AS total_impressions,
    SUM(clicks)                                             AS total_clicks,
    ROUND(SUM(clicks) * 1.0 / NULLIF(SUM(impressions), 0), 4) AS overall_ctr,
    SUM(cost)                                               AS total_cost,
    SUM(conversions)                                        AS total_conversions,
    ROUND(SUM(conversions) / NULLIF(SUM(clicks), 0), 4)     AS overall_cvr,
    ROUND(SUM(cost) / NULLIF(SUM(conversions), 0), 2)       AS overall_cpa,
    ROUND(SUM(cost) / NULLIF(SUM(clicks), 0), 2)            AS overall_cpc,
    COUNT(DISTINCT keyword_clean)                            AS keyword_count,
    SUM(CASE WHEN is_wasted_spend = 1 THEN cost ELSE 0 END) AS wasted_spend
FROM analytics_keyword
GROUP BY campaign_label, fiscal_year, program_type;


-- 4b. Ad group performance summary
CREATE VIEW vw_adgroup_summary AS
SELECT
    ad_group,
    SUM(impressions)                                        AS total_impressions,
    SUM(clicks)                                             AS total_clicks,
    SUM(conversions)                                        AS total_conversions,
    SUM(cost)                                               AS total_cost,
    ROUND(SUM(conversions) / NULLIF(SUM(clicks), 0), 4)     AS cvr,
    ROUND(SUM(cost) / NULLIF(SUM(clicks), 0), 2)            AS avg_cpc,
    ROUND(SUM(cost) / NULLIF(SUM(conversions), 0), 2)       AS cpa,
    SUM(CASE WHEN is_wasted_spend = 1 THEN cost ELSE 0 END) AS wasted_spend,

    -- Actionable recommendation flag
    CASE
        WHEN SUM(conversions) = 0 AND SUM(cost) > 100       THEN 'PAUSE'
        WHEN SUM(cost) / NULLIF(SUM(conversions), 0) > 150  THEN 'PAUSE'
        WHEN SUM(cost) / NULLIF(SUM(conversions), 0) > 80   THEN 'OPTIMIZE'
        WHEN SUM(conversions) / NULLIF(SUM(clicks), 0) > 0.10 THEN 'SCALE'
        ELSE 'MONITOR'
    END                                                     AS action_flag

FROM analytics_keyword
GROUP BY ad_group
ORDER BY total_conversions DESC;


-- 4c. Keyword performance ranked
CREATE VIEW vw_keyword_ranked AS
WITH keyword_agg AS (
    SELECT
        keyword_clean,
        ad_group,
        campaign_label,
        match_type,
        SUM(impressions)                                    AS total_impressions,
        SUM(clicks)                                         AS total_clicks,
        SUM(conversions)                                    AS total_conversions,
        SUM(cost)                                           AS total_cost,
        ROUND(SUM(conversions) / NULLIF(SUM(clicks), 0), 4) AS cvr,
        ROUND(SUM(cost) / NULLIF(SUM(conversions), 0), 2)   AS cpa,
        ROUND(SUM(cost) / NULLIF(SUM(clicks), 0), 2)        AS avg_cpc,
        AVG(CASE WHEN quality_score IS NOT NULL
                 THEN quality_score END)                     AS avg_quality_score
    FROM analytics_keyword
    GROUP BY keyword_clean, ad_group, campaign_label, match_type
)
SELECT
    *,

    -- Rank by conversion volume
    ROW_NUMBER() OVER (ORDER BY total_conversions DESC, cvr DESC)
                                                            AS rank_by_conversions,

    -- Rank by efficiency (CPA) among converters only
    ROW_NUMBER() OVER (
        PARTITION BY CASE WHEN total_conversions > 0 THEN 1 ELSE 0 END
        ORDER BY cpa ASC
    )                                                       AS rank_by_efficiency,

    -- Flag anomalies: CVR > 50% with decent volume could be tracking issue
    CASE
        WHEN cvr > 0.50 AND total_clicks >= 5 THEN 'VERIFY_TRACKING'
        WHEN cvr > 0.10 AND cpa < 30          THEN 'HIGH_PERFORMER'
        WHEN total_impressions > 100
             AND total_conversions = 0
             AND total_cost > 50              THEN 'NEGATIVE_KW_CANDIDATE'
        ELSE NULL
    END                                                     AS anomaly_flag

FROM keyword_agg;


-- 4d. Match type comparison
CREATE VIEW vw_match_type_analysis AS
SELECT
    campaign_label,
    match_type,
    COUNT(DISTINCT keyword_clean)                           AS keyword_count,
    SUM(impressions)                                        AS total_impressions,
    SUM(clicks)                                             AS total_clicks,
    SUM(conversions)                                        AS total_conversions,
    SUM(cost)                                               AS total_cost,
    ROUND(SUM(conversions) / NULLIF(SUM(clicks), 0), 4)     AS cvr,
    ROUND(SUM(cost) / NULLIF(SUM(conversions), 0), 2)       AS cpa,
    ROUND(SUM(cost) / NULLIF(SUM(clicks), 0), 2)            AS avg_cpc
FROM analytics_keyword
GROUP BY campaign_label, match_type
ORDER BY campaign_label, match_type;


-- 4e. Wasted spend analysis — keywords costing money with 0 conversions
CREATE VIEW vw_wasted_spend AS
SELECT
    keyword_clean,
    campaign_label,
    ad_group,
    match_type,
    impressions,
    clicks,
    cost                                                    AS wasted_cost,
    avg_cpc,
    'Add as negative keyword or pause' AS recommendation
FROM analytics_keyword
WHERE is_wasted_spend = 1
  AND cost > 20
ORDER BY cost DESC;


-- 4f. Demographic cross-analysis (age × campaign)
CREATE VIEW vw_demo_age_performance AS
SELECT
    age_group,
    campaign_label,
    SUM(clicks)                                             AS total_clicks,
    SUM(impressions)                                        AS total_impressions,
    SUM(conversions)                                        AS total_conversions,
    SUM(cost)                                               AS total_cost,
    ROUND(SUM(conversions) / NULLIF(SUM(clicks), 0), 4)     AS cvr,
    ROUND(SUM(cost) / NULLIF(SUM(conversions), 0), 2)       AS cpa,

    -- Share of total conversions
    ROUND(SUM(conversions) * 100.0 / NULLIF(
        SUM(SUM(conversions)) OVER (), 0
    ), 1)                                                   AS pct_of_total_conversions

FROM clean_demo_age
GROUP BY age_group, campaign_label
ORDER BY total_conversions DESC;


-- ============================================================================
-- STEP 5: VALIDATION QUERIES
-- ============================================================================

-- 5a. Verify totals match Google Ads UI
SELECT
    'Campaign Totals Check' AS validation,
    SUM(total_impressions)  AS sum_impressions,  -- expect: 49,593
    SUM(total_clicks)       AS sum_clicks,        -- expect: 967
    SUM(total_cost)         AS sum_cost,          -- expect: 3,207.14
    SUM(total_conversions)  AS sum_conversions    -- expect: 73
FROM vw_campaign_summary;


-- 5b. Check for orphaned keywords (in keyword table but not in ad group report)
SELECT k.keyword_clean, k.ad_group
FROM analytics_keyword k
LEFT JOIN stg_adgroup_report a
    ON TRIM(a.ad_group) = k.ad_group
   AND TRIM(a.campaign) = k.campaign_label
WHERE a.ad_group IS NULL;


-- 5c. Identify data quality flags
SELECT
    SUM(CASE WHEN quality_score IS NULL THEN 1 ELSE 0 END) AS missing_quality_score,
    SUM(CASE WHEN ctr IS NULL THEN 1 ELSE 0 END)          AS missing_ctr,
    SUM(CASE WHEN impressions = 0 THEN 1 ELSE 0 END)      AS zero_impression_rows,
    SUM(CASE WHEN is_wasted_spend = 1 THEN 1 ELSE 0 END)  AS wasted_spend_rows,
    SUM(CASE WHEN is_wasted_spend = 1 THEN cost ELSE 0 END) AS total_wasted_dollars,
    COUNT(*)                                                AS total_rows
FROM analytics_keyword;
