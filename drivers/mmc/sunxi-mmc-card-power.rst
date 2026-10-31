.. SPDX-License-Identifier: GPL-2.0

Card power cycling before initialization
=======================================

The optional boolean ``allwinner,card-power-cycle-on-power-up`` changes the
existing ``card-pwr-gpios`` AUTO policy for boards that must retain card power
during suspend and reset the card supply before reinitialization.

The property requires a valid ``card-pwr-gpios`` GPIO and both ``default`` and
``power-cycle`` pinctrl states. The GPIO's standard active-low flag determines
its polarity; the driver requests it initially ON and synchronizes the existing
``CARD_PWR_GPIO_HIGH_ACTIVE`` capability with that polarity. The ``power-cycle``
state must isolate all card signal pins as inputs with bias disabled to prevent
back-powering the card while its supply is OFF.

On each actual MMC_POWER_UP transition in AUTO mode, the driver selects
``power-cycle``, switches the GPIO OFF for ``time_pwroff_ms`` (default 200 ms),
switches it ON, waits ``post-power-on-delay-ms`` (default 10 ms), and restores
``default`` before initializing the controller. The MMC core additionally waits
that same power-on delay before and after enabling the initialization clock.

With this property, MMC_POWER_OFF and the driver's manual-stop error path do not
switch the GPIO OFF. The existing sleep/UART pinctrl sequence, regulators, SD
command sequence, and UHS capabilities are unchanged. The MANUAL debug mode leaves
GPIO control to its existing sysfs interface. Without this property, the legacy
AUTO and MANUAL policies are unchanged.

Only use this policy with a dedicated card power switch. It does not control a
shared upstream regulator or provide general hotplug support.
