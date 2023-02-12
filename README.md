-- # SQL-covid-project
select *
from CovidDeaths
where continent is not null
order by 3,4

-- showing death rate for Nigeria

select location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 DeathPercentage
from PortfolioProject..CovidDeaths
where location like 'nigeria'
order by DeathPercentage desc

-- showing infection rates for Nigeria

select location, date, population, total_cases, (total_cases/population)*100 InfectedPercentage
from PortfolioProject..CovidDeaths
where location = 'nigeria'
order by 1,2

-- looking at countries with the highest infection per population

Select location, population, max(total_cases) as InfectionHigh, max((total_cases/population)*100) MaxInfectionRate
from CovidDeaths
group by location, population
order by MaxInfectionRate desc

-- showing the countries with the higest death count per population

select location, max(cast(total_deaths as int)) as TotalDeathCount
from CovidDeaths
where continent is not null
group by location
order by TotalDeathCount desc


-- BREAKING THINGS DOWN BY CONTINENT

-- Showing continents with the highest death counts

Select continent, MAX(cast(Total_deaths as int)) as TotalDeathCount
From PortfolioProject..CovidDeaths
Where continent is not null 
Group by continent
order by TotalDeathCount desc

-- GLOBAL NUMBERS

Select sum(new_cases) Total_Cases, sum(cast(new_deaths as int)) Total_Deaths, sum(cast(new_deaths as int))/sum(new_cases)*100 DeathPercentage
from PortfolioProject..CovidDeaths
--group by date

--Looking At Vaccinations

select *
from CovidVaccinations

--Looking at Total Population Vs Vaccination

select dea.continent, dea.location, dea.date, dea.Population, vac.new_vaccinations
, Sum(convert(int,vac.new_vaccinations)) over (partition by dea.location order by dea.date) RollingPeopleVaccinated

from PortfolioProject..CovidDeaths dea
join PortfolioProject..CovidVaccinations vac
	on dea.date = vac.date
	and dea.location = vac.location
where dea.continent is not null
order by 2,3



-- USE CTE

with PopvsVacc (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(select dea.continent, dea.location, dea.date, dea.Population, vac.new_vaccinations
, Sum(convert(int,vac.new_vaccinations)) over (partition by dea.location order by dea.date) RollingPeopleVaccinated
from PortfolioProject..CovidDeaths dea
join PortfolioProject..CovidVaccinations vac
	on dea.date = vac.date
	and dea.location = vac.location
where dea.continent is not null
--order by 2,3
)
Select *,(Total_Vaccinations/Population)*100 PercentageVaccinated
From PopvsVacc


--Showing Max Percentage Vaccinated


With  PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(select dea.continent, dea.location, dea.date, dea.Population, vac.new_vaccinations
, Sum(convert(int,vac.new_vaccinations)) over (partition by dea.location order by dea.date) RollingPeopleVaccinated
from PortfolioProject..CovidDeaths dea
join PortfolioProject..CovidVaccinations vac
	on dea.date = vac.date
	and dea.location = vac.location
where dea.continent is not null
group by dea.continent, dea.location, dea.date, population, new_vaccinations
)
Select *, max((Total_Vaccinations/Population)*100) over (partition by location) MaxPercVaccinated
From PopvsVac



--TEMP TABLE

Drop Table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_Vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
select dea.continent, dea.location, dea.date, dea.Population, vac.new_vaccinations
, Sum(convert(int,vac.new_vaccinations)) over (partition by dea.location order by dea.date) RollingPeopleVaccinated
from PortfolioProject..CovidDeaths dea
join PortfolioProject..CovidVaccinations vac
	on dea.date = vac.date
	and dea.location = vac.location
--where dea.continent is not null

select * 
from #PercentPopulationVaccinated
order by 2,3


-- Creating View to store data for later visualizations

Create view PercentPopulationVaccinated as
select dea.continent, dea.location, dea.date, dea.Population, vac.new_vaccinations
, Sum(convert(int,vac.new_vaccinations)) over (partition by dea.location order by dea.date) RollingPeopleVaccinated
from PortfolioProject..CovidDeaths dea
join PortfolioProject..CovidVaccinations vac
	on dea.date = vac.date
	and dea.location = vac.location
where dea.continent is not null
--order by 2,3


Select *
From PercentPopulationVaccinated
