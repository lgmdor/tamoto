<script>
import SettingsTime from "$lib/SettingsTime.svelte";
import SettingsGeneral from "$lib/SettingsGeneral.svelte";
import { Tabs, TabsMenu, TabsMenuItem, TabsWrapper } from "$lib/Tabs/tabs.js";
import IconTime from "$icons/time.svg?component";
import IconSettings from "$icons/settings.svg?component";
import IconColor from "$icons/color.svg?component";
import IconInfo from "$icons/info.svg?component";
</script>

<!--------markup-------->

<div class="settings">
	<Tabs>
		<TabsWrapper>
			<SettingsTime />
		</TabsWrapper>
		<TabsWrapper>
			<SettingsGeneral />
		</TabsWrapper>

		<TabsMenu slot="menu">
			<TabsMenuItem tooltip="Time">
				<IconTime />
			</TabsMenuItem>
			<TabsMenuItem tooltip="Settings">
				<IconSettings />
			</TabsMenuItem>
			<TabsMenuItem tooltip="Themes">
				<IconColor />
			</TabsMenuItem>
			<TabsMenuItem tooltip="Info">
				<IconInfo />
			</TabsMenuItem>
		</TabsMenu>
	</Tabs>
</div>

<!--------style-------->
<style lang="sass">
	@use 'src/vars'
	@use 'src/oc'

	.settings
		position: relative
		width: 100%
		height: 100%
		padding-top: vars.$size-1
		padding-bottom: vars.$size-2

	:global(.settings .lgmui.tabs-menu-item)
		@include vars.bgHover(vars.$c-bg-3, 2%)
	:global(.settings .lgmui.tabs-menu)
		border-top: vars.$border-default
	:global(.settings .lgmui.tabs-wrapper)
		//padding-bottom: vars.$size-2
</style>
