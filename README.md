# COVID-Data-Explorartion

This repository will contain data exploration queries from the Our World in Data Dataset. Which can be found [here](https://ourworldindata.org/covid-deaths)

## Exploration Questions

- Select Data that we are going to be starting with

- Total Cases vs Total Deaths

- Total Cases vs Population. Shows what percentage of population infected with Covid

- Countries with Highest Infection Rate compared to Population

- Countries with Highest Death Count per Population

- Showing contintents with the highest death count per population

- Total Population vs Vaccinations (Shows Percentage of Population that has recieved at least one Covid Vaccine)

- Using CTE to perform Calculation on Partition By in previous query

- Creating View to store data for later visualizations

**Solution**

- Select Data that we are going to be starting with

``` SQL
Select Location, date, total_cases, new_cases, total_deaths, population
From covidedeaths
Where continent is not null
order by 1,2;
```

- Total Cases vs Total Deaths

``` SQL
Select Location, date, total_cases, total_deaths, ROUND((total_deaths/total_cases)*100, 2) as DeathPercentage
From coviddeaths
Where location like '%kenya%'
	and continent is not null
order by 1,2;
```

- Total Cases vs Population. Shows what percentage of population infected with Covid

``` SQL
Select Location, date, Population, total_cases, ROUND((total_cases/population)*100,2) as PercentPopulationInfected
From covidedeaths
order by 1,2;
```

- Countries with Highest Infection Rate compared to Population

``` SQL
Select Location, Population, MAX(total_cases) as HighestInfectionCount, Max((total_cases/population))*100 as PercentPopulationInfected
From covidedeaths
Group by Location, Population
order by PercentPopulationInfected desc;
```

- Countries with Highest Death Count per Population

``` SQL
Select Location, MAX(Total_deaths ) as TotalDeathCount
From covidedeaths
Where continent is not null
Group by Location
order by TotalDeathCount desc;
```
- Showing contintents with the highest death count per population

``` SQL
Select continent, MAX(Total_deaths ) as TotalDeathCount
From covidedeaths
Where continent is not null
Group by continent
order by TotalDeathCount desc;
```

- Global Numbers 

``` SQL
Select SUM(new_cases) as total_cases, SUM((new_deaths )) as total_deaths, SUM(new_deaths )/SUM(New_Cases)*100 as DeathPercentage
From covidedeaths
where continent is not null
order by 1,2;

- Total Population vs Vaccinations (Shows Percentage of Population that has recieved at least one Covid Vaccine)

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
From covidedeaths dea
	Join covidvaccinations vac
	On dea.location = vac.location
		and dea.date = vac.date
where dea.continent is not null
order by 2,3;
```

- Using CTE to perform Calculation on Partition By in previous query

``` SQL
With
	PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
	as
	(
		Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM((vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
		From covidedeaths dea
			Join covidvaccinations vac
			On dea.location = vac.location
				and dea.date = vac.date
		where dea.continent is not null
		order by 2,3
	)
Select *, (RollingPeopleVaccinated/Population)*100
From PopvsVac;
```

- Creating View to store data for later visualizations

``` SQL 
Create View PercentPopulationVacinated
as
	Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(vac.new_vaccinations) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
	From covidedeaths dea
		Join CovidVaccinations vac
		On dea.location = vac.location
			and dea.date = vac.date
	where dea.continent is not null;

SELECT *
FROM PercentPopulationVacinated
```




