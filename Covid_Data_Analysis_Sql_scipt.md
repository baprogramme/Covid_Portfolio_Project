---

# COVID-19 Data Exploration with SQL

This repository contains SQL queries used to explore a dataset of COVID-19 cases and vaccinations. The analysis aims to provide insights into the spread, mortality, and vaccination efforts related to the pandemic.

## Data Sources

The analysis utilizes two primary datasets:

*   **`CovidDeaths$`**: Contains information about COVID-19 cases, deaths, and location details.
*   **`CovidVaccinations$`**: Contains information about COVID-19 vaccinations administered, linked by location and date.

These datasets are assumed to be in a tabular format (e.g., CSV, Excel) and were likely imported into a SQL Server database.

## SQL Queries and Analysis

The following SQL queries were executed to perform various analyses:

### 1. Analyzing Total Cases vs. Total Deaths

*   **Purpose:** This query calculates the likelihood of dying if one contracts COVID-19 in a specific country.
*   **Filtering:** It specifically focuses on locations containing "states" in their name (likely to capture data at a more granular level within a larger country, though this might need clarification depending on your data) and excludes continent-level aggregations.
*   **Output:** Shows the location, date, total cases, total deaths, and the calculated death percentage.
*   **SQL Code:**

    ````sql
    ---Looking at Total Cases vs Total Deaths
    ---Shows likelihood of dying if you contract covid in your country

    select Location,date,total_cases,total_deaths,
    (total_deaths/total_cases)*100 as DeathPercentage from
    dbo.CovidDeaths$
    where location like '%states%' and continent is not null order by 1,2;
    ````

### 2. Analyzing Total Cases vs. Population

*   **Purpose:** This query determines the percentage of the population in different locations that have contracted COVID-19.
*   **Filtering:** It excludes continent-level aggregations and includes an optional filter (commented out) for locations containing "states".
*   **Output:** Displays the location, date, population, total cases, and the calculated percentage of the population infected.
*   **SQL Code:**

    ````sql
    ---Looking at Total Cases vs Population
    ---shows what percentage of population got Covid
    select Location,date,Population,total_cases,
    (total_cases/population)*100 as PercentPopulationInfected from
    dbo.CovidDeaths$
    where continent is not null
    --location like '%states%'
    order by 1,2;
    ````

### 3. Identifying Countries with the Highest Infection Rate Compared to Population

*   **Purpose:** This query identifies the countries with the highest percentage of their population infected with COVID-19.
*   **Aggregation:** It groups the data by location and population to find the maximum number of cases for each country.
*   **Filtering:** It excludes continent-level aggregations and includes an optional filter (commented out) for locations containing "states".
*   **Output:** Shows the location, population, the highest infection count recorded, and the corresponding percentage of the population infected, ordered by the infection percentage in descending order.
*   **SQL Code:**

    ````sql
    -- Looking at Countries with Highest Infection Rate compared to Population

    Select Location, Population, MAX(total_cases) as HighestInfectioncount,
    Max((total_cases/population))*100 as PercentPopulationInfected
    From PortfolioProject..CovidDeaths$
    where continent is not null
    --location like '%states%'
    group by Location, Population
    order by PercentPopulationInfected desc;
    ````

### 4. Showing Countries with the Highest Death Count per Population

*   **Purpose:** This query identifies the countries with the highest total number of deaths attributed to COVID-19.
*   **Data Type Conversion:** It explicitly casts the `Total_deaths` column to an integer for accurate aggregation.
*   **Filtering:** It excludes continent-level aggregations and includes an optional filter (commented out) for locations containing "states".
*   **Aggregation:** It groups the data by location and finds the maximum total death count for each country.
*   **Output:** Displays the location and the total death count, ordered by the death count in descending order.
*   **SQL Code:**

    ````sql
    ----showing countries with Highest Death count per Population

    select Location,MAX(Cast(Total_deaths as int)) as TotalDeathCount
    From PortfolioProject..CovidDeaths$
    where continent is not null
    -- and location like '%states%'
    Group by location
    order by TotalDeathCount desc
    ````

### 5 & 6. Analyzing Total Death Count by Continent

*   **Purpose:** These queries analyze the total number of deaths attributed to COVID-19, aggregated by continent.
*   **Data Type Conversion:** It explicitly casts the `Total_deaths` column to an integer.
*   **Filtering:** It specifically filters for rows where the `continent` is not null.
*   **Aggregation:** It groups the data by continent and finds the maximum total death count for each continent.
*   **Output:** Shows the continent and the total death count, ordered by the death count in descending order.
*   **Note:** The two identical queries suggest a potential redundancy or an intention to explore the same data point twice.
*   **SQL Code:**

    ````sql
    --- Let's break things down by Continent

    select continent,MAX(Cast(Total_deaths as int)) as TotalDeathCount
    From PortfolioProject..CovidDeaths$
    where continent is not null
    -- and location like '%states%'
    Group by continent
    order by TotalDeathCount desc


    --- showing continents with the highest death count per population

    select continent,MAX(Cast(Total_deaths as int)) as TotalDeathCount
    From PortfolioProject..CovidDeaths$
    where continent is not null
    -- and location like '%states%'
    Group by continent
    order by TotalDeathCount desc
    ````

### 7. Analyzing Global Numbers

*   **Purpose:** This query calculates the total new cases, total new deaths, and the death percentage globally for each date.
*   **Aggregation:** It groups the data by date and sums the `new_cases` and `new_deaths` across all locations.
*   **Filtering:** It excludes rows where the `continent` is null, ensuring only global figures are considered.
*   **Output:** Shows the date, total new cases, total new deaths, and the calculated death percentage for that day.
*   **SQL Code:**

    ````sql
    ---- GLOBAL NUMBERS

    select date,SUM(new_cases) as total_new_cases,SUM(cast(new_deaths as int)) as total_new_deaths,
    SUM(cast(new_deaths as int))/SUM(New_cases)*100 as DeathPercentage
    --Location,date,total_cases,total_deaths,
    --(total_deaths/total_cases)*100 as DeathPercentage
    from dbo.CovidDeaths$
    --where location like '%states%' and
    where continent is not null
    Group by date
    order by 1,2;
    ````

### 8. Joining Deaths and Vaccinations Data

*   **Purpose:** This query joins the `CovidDeaths$` and `CovidVaccinations$` tables based on location and date to explore vaccination data in relation to death data.
*   **Rolling Calculation:** It calculates the cumulative number of people vaccinated over time for each location using a window function (`SUM(...) OVER (PARTITION BY ... ORDER BY ...)`).
*   **Filtering:** It excludes rows where the `continent` is null.
*   **Output:** Shows the continent, location, population, new vaccinations for each day, and the rolling count of people vaccinated.
*   **SQL Code:**

    ````sql
    select dea.continent,dea.location,dea.population,vac.new_vaccinations
    ,SUM(Cast(vac.new_vaccinations as int)) over (partition by dea.location order by dea.location
    ,dea.date) as RollingPeopleVaccinated
    from PortfolioProject..CovidDeaths$ dea
    join PortfolioProject..CovidVaccinations$ vac
        on dea.location=vac.location
        and dea.date=vac.date
    where dea.continent is not null
    order by 2,3
    ;
    ````

### 9. Using Common Table Expression (CTE) for Further Analysis

*   **Purpose:** This section demonstrates the use of a CTE to calculate the percentage of the population that has been vaccinated.
*   **CTE Definition:** The `PopvsVac` CTE encapsulates the joined data and the rolling vaccination count from the previous query.
*   **Final Selection:** The final query selects all columns from the CTE and calculates the percentage of the population vaccinated by dividing the `RollingPeopleVaccinated` by the `Population`.
*   **SQL Code:**

    ````sql
    ---USE CTE
    with PopvsVac (continent, Location, Date, Population, New_vaccinations, RollingPeoplevaccinated)
    as (
    select dea.continent,dea.location,dea.date,dea.population,vac.new_vaccinations
    ,SUM(Cast(vac.new_vaccinations as int)) over (partition by dea.location order by dea.location
    ,dea.date) as RollingPeopleVaccinated
    from PortfolioProject..CovidDeaths$ dea
    join PortfolioProject..CovidVaccinations$ vac
        on dea.location=vac.location
        and dea.date=vac.date
    where dea.continent is not null
    --order by 2,3
    )
    select *, (RollingPeoplevaccinated/Population)*100 from PopvsVac
    ;
    ````

### 10. Using a Temporary Table for Further Analysis

*   **Purpose:** This section demonstrates the use of a temporary table (`#PercentPopulationVaccinated`) to achieve the same result as the CTE, calculating the percentage of the population vaccinated.
*   **Temporary Table Creation:** A temporary table with the necessary columns is created.
*   **Data Insertion:** The results of the joined query with the rolling vaccination count are inserted into the temporary table.
*   **Final Selection:** The final query selects all columns from the temporary table and calculates the percentage of the population vaccinated.
*   **Note:** The `DROP TABLE IF EXISTS` statement ensures that if a table with the same name exists, it is dropped before creating a new one.
*   **SQL Code:**

    ````sql
    --TEMP TABLE
    Drop table if exists #PercentPopulationVaccinated
    Create Table #PercentPopulationVaccinated
    (
    continent nvarchar(255),
    Location nvarchar(255),
    Date datetime,
    Population numeric,
    New_vaccinations numeric,
    RollingPeopleVaccinated numeric
    )

    Insert into #PercentPopulationVaccinated
    select dea.continent,dea.location,dea.date,dea.population,vac.new_vaccinations
    ,SUM(Cast(vac.new_vaccinations as int)) over (partition by dea.location order by dea.location
    ,dea.date) as RollingPeopleVaccinated
    from PortfolioProject..CovidDeaths$ dea
    join PortfolioProject..CovidVaccinations$ vac
        on dea.location=vac.location
        and dea.date=vac.date
    --where dea.continent is not null
    --order by 2,3


    select *,(RollingPeopleVaccinated/Population)*100
    From #PercentPopulationVaccinated;
    ````

### 11. Creating a View for Later Visualizations

*   **Purpose:** This section demonstrates the creation of a SQL view (`PercentPopulationVaccinated`) to store the result of the population vaccination percentage calculation. Views can be useful for simplifying future queries and for use in visualization tools.
*   **View Definition:** The `CREATE VIEW` statement defines the view based on the joined data and the rolling vaccination count.
*   **SQL Code:**

    ````sql
    ----Creating view to store data for later visualizations

    Create view PercentPopulationVaccinated as
    select dea.continent,dea.location,dea.date,dea.population,vac.new_vaccinations
    ,SUM(Cast(vac.new_vaccinations as int)) over (partition by dea.location order by dea.location
    ,dea.date) as RollingPeopleVaccinated
    from PortfolioProject..CovidDeaths$ dea
    join PortfolioProject..CovidVaccinations$ vac
        on dea.location=vac.location
        and dea.date=vac.date
    where dea.continent is not null
    --order by 2,3 and two source files Covideath.xlsx as well as Covid_vaccinations.xlsx
    ````

## Next Steps

The results of these queries can be further utilized for:

*   **Data Visualization:** Creating charts and graphs to better understand the trends and patterns in the data.
*   **Further Analysis:** Conducting more in-depth investigations based on the initial findings.
*   **Reporting:** Summarizing the key insights from the data exploration.

---

