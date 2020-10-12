# デフォルトの設定ファイル

## model.yaml

```yaml
import:  # Import other files from paths relative to this file, or absolute paths
    - 'model_config/techs.yaml'  # This file specifies the model's technologies
    - 'model_config/locations.yaml'  # This file specifies the model's locations
    - 'scenarios.yaml'  # Scenario and override group definitions

# Model configuration: all settings that affect the built model
model:
    name: National-scale example model

    # What version of Calliope this model is intended for
    calliope_version: 0.6.5

    # Time series data path - can either be a path relative to this file, or an absolute path
    timeseries_data_path: 'timeseries_data'

    subset_time: ['2005-01-01', '2005-01-05']  # Subset of timesteps

# Run configuration: all settings that affect how the built model is run
run:
    solver: cbc

    ensure_feasibility: true  # Switches on the "unmet demand" constraint

    bigM: 1e6  # Sets the scale of unmet demand, which cannot be too high, otherwise the optimisation will not converge

    zero_threshold: 1e-10  # Any value coming out of the backend that is smaller than this (due to floating point errors, probably) will be set to zero

    mode: plan  # Choices: plan, operate

    objective_options.cost_class: {monetary: 1}  

```

## locations.yaml

```yaml
##
# LOCATIONS
##

locations:
    # region-1-start
    region1:
        coordinates: {lat: 40, lon: -2}
        techs:
            demand_power:
                constraints:
                    resource: file=demand-1.csv:demand
            ccgt:
                constraints:
                    energy_cap_max: 30000 # increased to ensure no unmet_demand in first timestep
    # region-1-end
    # other-locs-start
    region2:
        coordinates: {lat: 40, lon: -8}
        techs:
            demand_power:
                constraints:
                    resource: file=demand-2.csv:demand
            battery:

    region1-1.coordinates: {lat: 41, lon: -2}
    region1-2.coordinates: {lat: 39, lon: -1}
    region1-3.coordinates: {lat: 39, lon: -2}

    region1-1, region1-2, region1-3:
        techs:
            csp:
    # other-locs-end

##
# TRANSMISSION CAPACITIES
##

links:
    # links-start
    region1,region2:
        techs:
            ac_transmission:
                constraints:
                    energy_cap_max: 10000
    region1,region1-1:
        techs:
            free_transmission:
    region1,region1-2:
        techs:
            free_transmission:
    region1,region1-3:
        techs:
            free_transmission:
    # links-end
```

## techs.yaml

```yaml
##
# TECHNOLOGY DEFINITIONS
##

# Note: '-start' and '-end' is used in tutorial documentation only

techs:

    ##
    # Supply
    ##

    # ccgt-start
    ccgt:
        essentials:
            name: 'Combined cycle gas turbine'
            color: '#E37A72'
            parent: supply
            carrier_out: power
        constraints:
            resource: inf
            energy_eff: 0.5
            energy_cap_max: 40000  # kW
            energy_cap_max_systemwide: 100000  # kW
            energy_ramping: 0.8
            lifetime: 25
        costs:
            monetary:
                interest_rate: 0.10
                energy_cap: 750  # USD per kW
                om_con: 0.02  # USD per kWh
    # ccgt-end

    # csp-start
    csp:
        essentials:
            name: 'Concentrating solar power'
            color: '#F9CF22'
            parent: supply_plus
            carrier_out: power
        constraints:
            storage_cap_max: 614033
            energy_cap_per_storage_cap_max: 1
            storage_loss: 0.002
            resource: file=csp_resource.csv
            resource_unit: energy_per_area
            energy_eff: 0.4
            parasitic_eff: 0.9
            resource_area_max: inf
            energy_cap_max: 10000
            lifetime: 25
        costs:
            monetary:
                interest_rate: 0.10
                storage_cap: 50
                resource_area: 200
                resource_cap: 200
                energy_cap: 1000
                om_prod: 0.002
    # csp-end

    ##
    # Storage
    ##
    # battery-start
    battery:
        essentials:
            name: 'Battery storage'
            color: '#3B61E3'
            parent: storage
            carrier: power
        constraints:
            energy_cap_max: 1000  # kW
            storage_cap_max: inf
            energy_cap_per_storage_cap_max: 4
            energy_eff: 0.95  # 0.95 * 0.95 = 0.9025 round trip efficiency
            storage_loss: 0  # No loss over time assumed
            lifetime: 25
        costs:
            monetary:
                interest_rate: 0.10
                storage_cap: 200  # USD per kWh storage capacity
    # battery-end

    ##
    # Demand
    ##
    # demand-start
    demand_power:
        essentials:
            name: 'Power demand'
            color: '#072486'
            parent: demand
            carrier: power
    # demand-end

    ##
    # Transmission
    ##

    # transmission-start
    ac_transmission:
        essentials:
            name: 'AC power transmission'
            color: '#8465A9'
            parent: transmission
            carrier: power
        constraints:
            energy_eff: 0.85
            lifetime: 25
        costs:
            monetary:
                interest_rate: 0.10
                energy_cap: 200
                om_prod: 0.002

    free_transmission:
        essentials:
            name: 'Local power transmission'
            color: '#6783E3'
            parent: transmission
            carrier: power
        constraints:
            energy_cap_max: inf
            energy_eff: 1.0
        costs:
            monetary:
                om_prod: 0
    # transmission-end

```