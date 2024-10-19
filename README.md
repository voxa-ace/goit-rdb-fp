# goit-rdb-fp

# Task 1

CREATE SCHEMA IF NOT EXISTS pandemic;

USE pandemic;

# Task 2

CREATE TABLE entities (
    id INT AUTO_INCREMENT PRIMARY KEY,
    entity_name VARCHAR(255),
    code VARCHAR(10),
    UNIQUE(entity_name, code)
);

CREATE TABLE IF NOT EXISTS infectious_cases (
    Entity VARCHAR(255),
    Code VARCHAR(10),
    Year INT,
    Number_yaws INT,
    polio_cases INT,
    cases_guinea_worm INT,
    Number_rabies INT,
    Number_malaria INT,
    Number_hiv INT,
    Number_tuberculosis INT,
    Number_smallpox INT,
    Number_cholera_cases INT
);

INSERT INTO entities (entity_name, code)
SELECT DISTINCT Entity, Code FROM infectious_cases;

SELECT * FROM entities;


CREATE TABLE IF NOT EXISTS normalized_infectious_cases (
    id INT AUTO_INCREMENT PRIMARY KEY,
    entity_id INT,
    year INT,
    number_yaws INT,
    polio_cases INT,
    cases_guinea_worm INT,
    number_rabies INT,
    number_malaria INT,
    number_hiv INT,
    number_tuberculosis INT,
    number_smallpox INT,
    number_cholera_cases INT,
    FOREIGN KEY (entity_id) REFERENCES entities(id)
);

INSERT INTO normalized_infectious_cases (
    entity_id, year, number_yaws, polio_cases, cases_guinea_worm, 
    number_rabies, number_malaria, number_hiv, number_tuberculosis, 
    number_smallpox, number_cholera_cases
)
SELECT 
    e.id, ic.Year, 
    NULLIF(ic.Number_yaws, '') AS number_yaws, 
    NULLIF(ic.polio_cases, '') AS polio_cases, 
    NULLIF(ic.cases_guinea_worm, '') AS cases_guinea_worm, 
    NULLIF(ic.Number_rabies, '') AS number_rabies, 
    NULLIF(ic.Number_malaria, '') AS number_malaria, 
    NULLIF(ic.Number_hiv, '') AS number_hiv, 
    NULLIF(ic.Number_tuberculosis, '') AS number_tuberculosis, 
    NULLIF(ic.Number_smallpox, '') AS number_smallpox, 
    NULLIF(ic.Number_cholera_cases, '') AS number_cholera_cases
FROM infectious_cases ic
JOIN entities e ON ic.Entity = e.entity_name AND ic.Code = e.code;


# Task 3

SELECT 
    e.entity_name, 
    e.code, 
    AVG(nic.number_rabies) AS avg_rabies, 
    MIN(nic.number_rabies) AS min_rabies, 
    MAX(nic.number_rabies) AS max_rabies, 
    SUM(nic.number_rabies) AS sum_rabies
FROM normalized_infectious_cases nic
JOIN entities e ON nic.entity_id = e.id
WHERE nic.number_rabies IS NOT NULL
GROUP BY e.entity_name, e.code
ORDER BY avg_rabies DESC
LIMIT 10;


# Task 4

SELECT 
    year,
    CONCAT(year, '-01-01') AS first_january,
    CURDATE() AS current_date,
    TIMESTAMPDIFF(YEAR, CONCAT(year, '-01-01'), CURDATE()) AS years_difference
FROM normalized_infectious_cases;


# Task 5

DELIMITER //

CREATE FUNCTION cases_per_period(cases_year INT, period INT) 
RETURNS FLOAT
DETERMINISTIC
BEGIN
    RETURN cases_year / period;
END //

DELIMITER ;

SELECT 
    number_rabies,
    cases_per_period(number_rabies, 4) AS rabies_per_quarter
FROM normalized_infectious_cases
WHERE number_rabies IS NOT NULL;
