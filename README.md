# MonkeyPox-Exploratory-Analysis
The series of query explores the Monkey Pox Virus and its impact on world health
Select * 
FROM ['Monkey Pox Cases (World Wide) S$']


Select * 
FROM ['Monkey Pox Cases (World Wide) S$']
where location <> 'World'
order by 2;

---Select data to start analysis with
Select location,date,new_cases,total_cases,new_cases_per_million,total_cases_per_million, new_deaths, total_cases_per_million
FROM ['Monkey Pox Cases (World Wide) S$']
where location is not null
order by 1,2;

--List of countries affected by Monkey Pox
SELECT DISTINCT location AS 'List of Countries'
FROM ['Monkey Pox Cases (World Wide) S$']
WHERE total_cases IS NOT NULL and location <> 'World'
ORDER BY location;

---Number of countries affected by Monkey Pox
SELECT COUNT(*) As 'Number of Countries' FROM 
	(SELECT DISTINCT location AS 'List of Countries'
FROM ['Monkey Pox Cases (World Wide) S$']
WHERE total_cases IS NOT NULL and location <> 'World'
) As Country_count;

-- Date Of First Case Reported For Each Country 
SELECT location AS Country, MIN(Cast(date AS date)) AS FirstCaseReportedOn 
FROM ['Monkey Pox Cases (World Wide) S$']
WHERE Location <> 'World' and total_cases IS NOT NULL
GROUP BY location
ORDER BY 2;

-- Date Of First Case Reported For Nigeria 
SELECT location AS Country, MIN(Cast(date AS date)) AS FirstCaseReportedOn 
FROM ['Monkey Pox Cases (World Wide) S$']
WHERE Location = 'Nigeria' and total_cases IS NOT NULL
GROUP BY location
ORDER BY 2;


--Total Cases vs Total Deaths
-- This shows the likelihood of dying if you contract MonkeyPox in your country

Select Location, date, total_cases,total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From ['Monkey Pox Cases (World Wide) S$']
Where location like '%Nigeria%'
order by 1,2;

--- TOP 10 Countries with the Higest Infection Rate

Select TOP 10 Location, Max(total_cases) as HighestInfectionCount
From ['Monkey Pox Cases (World Wide) S$'] 
Where location <> 'World'
Group by Location
Order By HighestInfectionCount desc;

Select TOP 10 Location, Max(total_cases) as HighestInfectionCount
From ['Monkey Pox Cases (World Wide) S$'] 
Where location = 'Nigeria'
Group by Location
Order By HighestInfectionCount desc;



--Countries with Highest Death Rate
Select  TOP 5 Location, Max(total_deaths) as HighestDeathCount
From ['Monkey Pox Cases (World Wide) S$']
Where location <> 'World'
Group by location
Order by HighestDeathCount desc;

-- Global Numbers
Select Sum(new_cases) as total_cases, Sum(cast(new_deaths as int)) as total_deaths
FROM ['Monkey Pox Cases (World Wide) S$']
where location = 'World'
order by 1,2;

-- SUMMARY of cases : Isolating Total Cases, Total Deaths For Each Country
CREATE VIEW cases_summary AS
SELECT location AS Country,
MAX(total_cases) AS Total_Cases, 
MAX(total_deaths) AS Total_Deaths,
MAX(total_deaths)/MAX(total_cases)*100 AS Death_Percentage -- percentage of deaths out of total number of cases
FROM ['Monkey Pox Cases (World Wide) S$']
WHERE Location <> 'World' AND total_cases IS NOT NULL
GROUP BY location;

Select * from cases_summary
ORDER BY 2 DESC;

Select * from cases_summary where Country = 'Nigeria';


SELECT Top 10 Country, Total_Deaths
FROM cases_summary
ORDER BY Total_Deaths DESC;

Select date,location, Sum(new_cases) as total_cases, Sum(cast(new_deaths as int)) as total_deaths, 
Sum(cast(new_deaths as int))/Sum(new_cases) as DeathPercentage
FROM ['Monkey Pox Cases (World Wide) S$']
where location = 'World'
Group by date, location
order by 5 DESC;

Select Sum(new_cases) as total_cases, Sum(cast(new_deaths as int)) as total_deaths
FROM ['Monkey Pox Cases (World Wide) S$']
where location = 'World'
order by 1,2;

-- TO determine the Total death per location and sort by the highest death
Select location, total_deaths, total_cases, 
SUM(Convert(int, total_deaths)) Over (Partition by location) AS SUMDEATHS
FROM ['Monkey Pox Cases (World Wide) S$']
Where location <> 'World'
GROUP BY loCation,total_deaths, total_cases
order by 4 DESC;

--- To determine the total cases per location and sort by the highest cases
Select location, total_cases, total_deaths,
SUM(Convert(int, total_cases)) Over (Partition by location) AS ROLLINGSUMCASES
FROM ['Monkey Pox Cases (World Wide) S$']
Where location <> 'World'
GROUP BY loCation,total_deaths, total_cases
order by 4 DESC;

With DeathsVsCases (location, total_cases, total_deaths,ROLLINGSUMDEATHS)
as(
Select location, total_cases, total_deaths,
SUM(Convert(int, total_deaths)) Over (Partition by location) AS ROLLINGSUMDEATHS
FROM ['Monkey Pox Cases (World Wide) S$']
Where location <> 'World'
GROUP BY loCation,total_deaths, total_cases
)
Select *, (ROLLINGSUMDEATHS/total_cases)*100 AS Percentagedeaths
FROM DeathsVsCases
ORDER BY 5 DESC;








