This toggle switch does not have hardcoded heights/widths, so you can set those yourself on the custom-switch class. If you don't, it will take all available space.

The 2 states are labelled next (before and after) to the switch.

````html
<div class="custom-control custom-switch d-flex">
    <input type="checkbox" class="custom-control-input" id="switch" checked>
    <label for="switch" class="custom-control-label order-1"></label>

    <label for="switch" class="switch-off-label flex-grow-1 order-0">
        <span>
            Off
        </span>
    </label>
    <label for="switch" class="switch-on-label flex-grow-1 order-2">
        <span>
            On
        </span>
    </label>
</div>
````

````scss
// set border-radius with $custom-switch-indicator-border-radius in variables.scss
$switch-height: 32px !default;
$switch-width: 80px !default;
$switch-indicator-width: 24px !default;
$switch-indicator-clearance: 4px !default;

.custom-switch {
  --on-bg: #{$primary};
  --on-indicator-bg: #{$white};
  --off-bg: #{$light};
  --off-indicator-bg: #{$white};
  --focus-color: #{rgba($primary, 0.25)};

  padding-left: 0;

  .custom-control-label {
    margin: 0 20px;
    height: $switch-height;
    width: $switch-width;

    &:before {
      position: static;
      height: 100%;
      width: 100%;
      border: none;
    }

    &:after {
      top: $switch-indicator-clearance;
      left: $switch-indicator-clearance;
      height: calc(100% - #{$switch-indicator-clearance * 2});
      width: $switch-indicator-width;
    }
  }

  .custom-control-input {
    &:focus {
      ~ .custom-control-label {
        &:before {
          box-shadow: 0 0 0 0.2rem var(--focus-color);
        }
      }
    }

    &:checked {
      ~ .custom-control-label {
        &:before {
          background-color: var(--on-bg); // on state bg
        }

        &:after {
          transform: translateX(calc(#{$switch-width} - 100% - #{$switch-indicator-clearance * 2}));
          background-color: var(--on-indicator-bg);
        }
      }

      ~ .switch-on-label {
        font-weight: bold;
      }
    }

    &:not(:checked) {
      ~ .custom-control-label {
        &:before {
          background-color: var(--off-bg); // off state bg
        }

        &:after {
          background-color: var(--off-indicator-bg);
        }
      }

      ~ .switch-off-label {
        font-weight: bold;
      }
    }
  }

  .switch-on-label,
  .switch-off-label {
    position: relative;
    height: 1em;

    > span {
      position: absolute;
    }
  }

  .switch-off-label {
    > span {
      right: 0;
    }
  }
}
````

variables.scss (example)

````scss
$switch-height: 32px;
$switch-width: 80px;
$switch-indicator-width: 24px;
$switch-indicator-clearance: 4px;
$custom-switch-indicator-border-radius: 9999px;
````
