# Covid_Data_Exploration_using_SQL
Tools Used : SQL Server


Load all columns of CovidDeath table (continent = Not Null)
------------------------------------------------------------
Select *
From CovidData..Coviddeaths
Where continent is not null 
order by 3,4


Summary view of CovidDeath table
---------------------------------
Select Location, date, total_cases, new_cases, total_deaths, population
From CovidData..Coviddeaths
Where continent is not null 
order by 1,2

DeathPercent of all countries(Total Cases Vs Total Deaths)
-----------------------------------------------------------
Select Location, date, total_cases,total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From CovidData..Coviddeaths
and continent is not null 
order by 1,2

DeathPercent of country India (Total Cases Vs Total Deaths)
-------------------------------------------------------------
Select Location, date, total_cases,total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From CovidData..Coviddeaths
Where location = 'india'
and continent is not null 
order by 1,2

what percentage of population infected with Covid (Total cases Vs Population)
---------------------------------------------------
Select Location, date, Population, total_cases,  (total_cases/population)*100 as PercentPopulationInfected
From CovidData..Coviddeaths
order by 1,2

what percentage of Indian population infected with Covid (Total cases Vs Population)
-------------------------------------------------------------------------------
Select Location, date, Population, total_cases,  (total_cases/population)*100 as PercentPopulationInfected
From CovidData..Coviddeaths
where location = 'India'
order by 1,2

Countries with Highest Infection Rate compared to Population
-------------------------------------------------------------
Select Location, Population, MAX(total_cases) as HighestInfectionCount,  Max((total_cases/population))*100 as PopulationInfectedPercent
From CovidData..Coviddeaths
Group by Location, Population
order by PopulationInfectedPercent desc

Countries with Highest Death Count per Population
--------------------------------------------------
Select Location, MAX(cast(Total_deaths as int)) as TotalDeathCount
From CovidData..Coviddeaths
Where continent is not null 
Group by Location
order by TotalDeathCount desc


CONTINENT WISE DATA
-------------------
Contintents with the highest death count per population
---------------------------------------------------------
Select continent, MAX(cast(Total_deaths as int)) as TotalDeathCount
From covidData..Coviddeaths
Where continent is not null 
Group by continent
order by TotalDeathCount desc


GLOBAL DATA
---------------
Select SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths, SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
From CovidData..Coviddeaths
where continent is not null 
Group By date
order by 1,2

VACCINATION
-----------
Percentage of Population that has recieved at least one Covid Vaccine (Total Population vs Vaccinations)
--------------------------------------------------------------------------------------------------------
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From CovidData..Coviddeaths dea
Join CovidData..Covidvaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
order by 2,3


Using CTE to perform Calculation on Partition By in previous query
-------------------------------------------------------------------
With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From CovidData..Coviddeaths dea
Join CovidData..Covidvaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
--order by 2,3
)
Select *, (RollingPeopleVaccinated/Population)*100
From PopvsVac



Using Temp Table to perform Calculation on Partition By in previous query
---------------------------------------------------------------------------
DROP Table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From CovidData..Coviddeaths dea
Join CovidData..Covidvaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
--where dea.continent is not null 
--order by 2,3

Select *, (RollingPeopleVaccinated/Population)*100
From #PercentPopulationVaccinated




Creating View to store data for later visualizations
------------------------------------------------------

Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From CovidData..Coviddeaths dea
Join CovidData..Covidvaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 


#WAIT FOR MORE....(work in progress)

