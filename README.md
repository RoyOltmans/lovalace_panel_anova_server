# Anova Precision Cooker Lovelace Panel

## Description
This Lovelace panel allows you to control and monitor the Anova Precision Cooker directly from Home Assistant. It requires the [anova_server](https://github.com/RoyOltmans/anova_server) integration to establish communication with your cooker.

## Prerequisites
1. **Home Assistant** – Make sure you have a working Home Assistant instance.
2. **anova_server** – Install and configure the [anova_server](https://github.com/RoyOltmans/anova_server). Refer to its documentation for setup instructions. Once configured, it provides the necessary REST endpoints and sensors for this Lovelace panel to function.
3. **Required Entities & Scripts** – Ensure that your `sensor.anova_temperature`, `sensor.anova_target_temperature`, `sensor.anova_status`, `timer.anova_timer`, `input_number.anova_target_temperature_selector`, and `input_datetime.anova_time_picker` are set up. Additionally, you should have `script.start_anova_precision_cooker` and `script.stop_anova_precision_cooker` scripts that interact with the anova_server.

## Installation
1. **Lovelace Setup** – Navigate to the `config` directory of your Home Assistant setup.
2. Open the `ui-lovelace.yaml` file or edit via the UI under **Lovelace Dashboards**.
3. **Install necessary custom cards.** You can install them via HACS (recommended) or manually:

   **timer-bar-card**  
   - Repository: [github.com/rianadon/timer-bar-card](https://github.com/rianadon/timer-bar-card)
   - Manual installation:
     1. Download the latest `timer-bar-card.js` from the [Releases](https://github.com/rianadon/timer-bar-card/releases).
     2. Place it in your `config/www` folder (e.g., `config/www/timer-bar-card.js`).
     3. Add the resource to your Lovelace configuration:
        ```yaml
        resources:
          - url: /local/timer-bar-card.js
            type: module
        ```

   **slider-entity-row**  
   - Repository: [github.com/thomasloven/lovelace-slider-entity-row](https://github.com/thomasloven/lovelace-slider-entity-row)
   - Manual installation:
     1. Download `slider-entity-row.js` from the [Releases](https://github.com/thomasloven/lovelace-slider-entity-row/releases).
     2. Place it in your `config/www` folder.
     3. Add the resource to your Lovelace configuration:
        ```yaml
        resources:
          - url: /local/slider-entity-row.js
            type: module
        ```

   **time-picker-card**  
   - Repository: [github.com/GeorgeSG/lovelace-time-picker-card](https://github.com/GeorgeSG/lovelace-time-picker-card)
   - Manual installation:
     1. Download `time-picker-card.js` from the [Releases](https://github.com/GeorgeSG/lovelace-time-picker-card/releases).
     2. Place it in your `config/www` folder.
     3. Add the resource to your Lovelace configuration:
        ```yaml
        resources:
          - url: /local/time-picker-card.js
            type: module
        ```

   **apexcharts-card**  
   - Repository: [github.com/RomRider/apexcharts-card](https://github.com/RomRider/apexcharts-card)
   - Manual installation:
     1. Download `apexcharts-card.js` from the [Releases](https://github.com/RomRider/apexcharts-card/releases).
     2. Place it in your `config/www` folder.
     3. Add the resource to your Lovelace configuration:
        ```yaml
        resources:
          - url: /local/apexcharts-card.js
            type: module
        ```

4. **Add the following configuration** to your `ui-lovelace.yaml` (or via the UI). **This example includes two vertical stacks: one for the Anova controller and one for recipes, plus an ApexCharts card.**

   ```yaml
   title: Cooking
   path: cooking
   icon: mdi:chef-hat
   type: masonry
   cards:
     - type: vertical-stack
       cards:
         - type: markdown
           content: |
             # Sous Vide Anova Precision Cooker
             Select a recipe to automatically set the Anova Precision Cooker.
         - type: picture
           image: /local/assets/images/devices/anova_precision_sous_vide_wd_rt.png
         - type: gauge
           entity: sensor.anova_temperature
           name: Current Temperature
           min: 0
           max: 100
           severity:
             green: 50
             yellow: 60
             red: 80
         - type: horizontal-stack
           cards:
             - show_name: true
               show_icon: true
               type: button
               name: Start
               icon: mdi:play
               tap_action:
                 action: perform-action
                 perform_action: script.start_anova_precision_cooker
                 target: {}
             - show_name: true
               show_icon: true
               type: button
               name: Stop
               icon: mdi:stop
               tap_action:
                 action: perform-action
                 perform_action: script.stop_anova_precision_cooker
                 target: {}
         - type: entities
           entities:
             - entity: timer.anova_timer
               type: custom:timer-bar-card
               name: Cook Time
           state_color: false
           show_header_toggle: true
         - type: entities
           entities:
             - type: custom:slider-entity-row
               entity: input_number.anova_target_temperature_selector
               name: Set Temperature
               icon: mdi:thermometer
               toggle: false
               full_row: true
               show_icon: true
               hide_when_off: false
             - entity: sensor.anova_temperature
               name: Current Temperature
               icon: mdi:thermometer
             - entity: sensor.anova_target_temperature
               name: Target Temperature
               icon: mdi:thermometer-chevron-up
             - entity: sensor.anova_status
               name: Anova Status
               icon: mdi:chef-hat
         - type: custom:time-picker-card
           entity: input_datetime.anova_time_picker
           hour_mode: 24
           hour_step: 1
           minute_step: 5
           layout:
             hour_mode: double
             align_controls: center
             name: inside
             thin: true
           hide:
             seconds: false
             icon: false
           name: Anova Duration
     - type: vertical-stack
       cards:
         - type: markdown
           content: |
             # Sous Vide Recipes
             Select a recipe to automatically set the Anova Precision Cooker.
         - type: entities
           entities:
             - entity: input_select.anova_recipe
               name: Select Recipe
         - square: false
           type: grid
           columns: 2
           cards:
             - type: picture
               image: /local/assets/images/recipes/steak.png
               tap_action:
                 action: call-service
                 service: script.set_sous_vide_recipe
                 data:
                   temperature: 54.5
                   duration: 90
               name: Steak (Medium-Rare)
             - type: picture
               image: /local/assets/images/recipes/chicken.png
               tap_action:
                 action: call-service
                 service: script.set_sous_vide_recipe
                 data:
                   temperature: 63
                   duration: 120
               name: Chicken Breast
             - type: picture
               image: /local/assets/images/recipes/salmon.png
               tap_action:
                 action: call-service
                 service: script.set_sous_vide_recipe
                 data:
                   temperature: 50
                   duration: 45
               name: Salmon
             - type: picture
               image: /local/assets/images/recipes/eggs.png
               tap_action:
                 action: call-service
                 service: script.set_sous_vide_recipe
                 data:
                   temperature: 75
                   duration: 13
               name: Eggs (Soft-Boiled)
             - type: picture
               image: /local/assets/images/recipes/carrots.png
               tap_action:
                 action: call-service
                 service: script.set_sous_vide_recipe
                 data:
                   temperature: 84
                   duration: 60
               name: Glazed Carrots
             - type: picture
               image: /local/assets/images/recipes/halibut.png
               tap_action:
                 action: call-service
                 service: script.set_sous_vide_recipe
                 data:
                   temperature: 60
                   duration: 40
               name: Butter-Basted Halibut
             - type: picture
               image: /local/assets/images/recipes/duck.png
               tap_action:
                 action: call-service
                 service: script.set_sous_vide_recipe
                 data:
                   temperature: 74
                   duration: 2160
               name: Duck Confit
             - type: picture
               image: /local/assets/images/recipes/chicken_wing.png
               tap_action:
                 action: call-service
                 service: script.set_sous_vide_recipe
                 data:
                   temperature: 74
                   duration: 60
               name: Buffalo Chicken Wings
             - type: picture
               image: /local/assets/images/recipes/corn.png
               tap_action:
                 action: call-service
                 service: script.set_sous_vide_recipe
                 data:
                   temperature: 84
                   duration: 30
               name: Corn on the Cob
             - type: picture
               image: /local/assets/images/recipes/porkchop.png
               tap_action:
                 action: call-service
                 service: script.set_sous_vide_recipe
                 data:
                   temperature: 60
                   duration: 90
               name: Pork Chops
             - type: picture
               image: /local/assets/images/recipes/lamb.png
               tap_action:
                 action: call-service
                 service: script.set_sous_vide_recipe
                 data:
                   temperature: 55
                   duration: 1440
               name: Leg of Lamb
             - type: picture
               image: /local/assets/images/recipes/lobster.png
               tap_action:
                 action: call-service
                 service: script.set_sous_vide_recipe
                 data:
                   temperature: 60
                   duration: 45
               name: Lobster Tails
             - type: picture
               image: /local/assets/images/recipes/porkribs.png
               tap_action:
                 action: call-service
                 service: script.set_sous_vide_recipe
                 data:
                   temperature: 62.8
                   duration: 2880
               name: Beef Short Ribs
     - type: vertical-stack
       cards:
         - type: custom:apexcharts-card
           header:
             show: true
             show_states: true
             colorize_states: true
           series:
             - entity: sensor.anova_temperature
             - entity: sensor.anova_target_temperature
               type: column
   ```

5. **Save** the file and refresh your Lovelace UI.
6. Ensure that any referenced images and custom cards are correctly installed and accessible in the `/local/` directory.

## Required Sensors
- **`sensor.anova_temperature`** – Current temperature of the Anova Precision Cooker.
- **`sensor.anova_target_temperature`** – Target temperature set for the Anova.
- **`sensor.anova_status`** – Current operational status of the Anova Precision Cooker.
- **`timer.anova_timer`** – Timer for the sous vide cooking process.
- **`input_number.anova_target_temperature_selector`** – Temperature selection slider.
- **`input_datetime.anova_time_picker`** – Time picker for setting cook duration.
- **`input_select.anova_recipe`** – Optionally, an input select for choosing the recipe.
- **Scripts**: `script.set_sous_vide_recipe`, `script.start_anova_precision_cooker`, and `script.stop_anova_precision_cooker` to manage and automate your sous vide process.

## Sample Script for Starting the Anova Precision Cooker
Below is an example of how your `script.start_anova_precision_cooker` might look in `scripts.yaml`. It calls the `rest_command.start_anova_cooking` and then starts a timer based on the user-selected duration in `input_datetime.anova_time_picker`:

```yaml
description: Start Anova Precision Cooker
icon: mdi:chef-hat
alias: Start Anova Precision Cooker
sequence:
  - service: rest_command.start_anova_cooking
    data: {}
  - service: timer.start
    data:
      duration: >-
        {% set time_parts = states('input_datetime.anova_time_picker').split(':') %}
        {{ time_parts[0] | int }}:{{ time_parts[1] | int }}:{{ time_parts[2] | int }}
    target:
      entity_id: timer.anova_timer
```

## Sample Script for Stopping the Anova Precision Cooker
Below is an example of how your `script.stop_anova_precision_cooker` might look in `scripts.yaml`. It calls the `rest_command.stop_anova_cooking` and then finishes the timer:

```yaml
description: Stop Anova Precision Cooker
icon: mdi:chef-hat
alias: Stop Anova Precision Cooker
sequence:
  - service: rest_command.stop_anova_cooking
    data: {}
  - service: timer.finish
    data: {}
    target:
      entity_id: timer.anova_timer
```

## Usage
1. Make sure your `anova_server` is running and integrated properly with Home Assistant.
2. Navigate to the **Cooking** dashboard in Home Assistant.
3. Use the buttons and sliders to control the Anova Precision Cooker, select recipes, and monitor temperatures.
4. Press **Start** to begin cooking with the currently configured temperature and time, or **Stop** to halt the cooker.
5. Select a recipe to automatically adjust the target temperature and duration.

## Configuration
- Adjust the temperature settings and recipes as needed for your preferred cooking results.
- Customize the UI to your liking by rearranging cards or editing the `ui-lovelace.yaml`.

## Contributing
Contributions are welcome. Please open an issue or pull request if you have suggestions or improvements.

## License
Specify the license under which the project is distributed.

## Contact
Provide contact details for further questions or contributions.

