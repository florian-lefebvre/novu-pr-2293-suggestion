<script setup lang="ts">
import { ref, useSlots, computed, onMounted } from "vue";
import { css } from "@emotion/css";
import {
  getStyleByPath,
  getDefaultTheme,
  getDefaultBellColors,
  NotificationCenterContentWebComponent,
} from "@novu/notification-center";
import BellButton from "./BellButton.vue";
import type { NotificationCenterContentComponentProps } from "@novu/notification-center";

export type FloatingPlacement = "end" | "start";
export type FloatingSide = "top" | "right" | "bottom" | "left" | "auto";
export type FloatingPosition =
  | FloatingSide
  | `${FloatingSide}-${FloatingPlacement}`;
export interface NotificationCenterComponentProps {
  backendUrl?: NotificationCenterContentComponentProps["backendUrl"];
  socketUrl?: NotificationCenterContentComponentProps["socketUrl"];
  subscriberId?: NotificationCenterContentComponentProps["subscriberId"];
  applicationIdentifier: NotificationCenterContentComponentProps["applicationIdentifier"];
  subscriberHash?: NotificationCenterContentComponentProps["subscriberHash"];
  stores?: NotificationCenterContentComponentProps["stores"];
  tabs?: NotificationCenterContentComponentProps["tabs"];
  showUserPreferences?: NotificationCenterContentComponentProps["showUserPreferences"];
  popover?: {
    offset?: number;
    position?: FloatingPosition;
  };
  theme?: NotificationCenterContentComponentProps["theme"];
  styles?: NotificationCenterContentComponentProps["styles"];
  colorScheme?: NotificationCenterContentComponentProps["colorScheme"];
  i18n?: NotificationCenterContentComponentProps["i18n"];
  sessionLoaded?: NotificationCenterContentComponentProps["sessionLoaded"];
  notificationClicked?: NotificationCenterContentComponentProps["notificationClicked"];
  unseenCountChanged?: NotificationCenterContentComponentProps["unseenCountChanged"];
  actionClicked?: NotificationCenterContentComponentProps["actionClicked"];
  tabClicked?: NotificationCenterContentComponentProps["tabClicked"];
}

customElements.define(
  "notification-center-content-component",
  NotificationCenterContentWebComponent
);

const props = withDefaults(defineProps<NotificationCenterComponentProps>(), {
  colorScheme: "dark",
});

const slots = useSlots();
const popper = ref(null);

const unseenCount = ref<number | undefined>(undefined);
const hasSlot = ref(!!slots.default);

onMounted(() => {
  const arrowsContainer = (popper.value as any).$_arrowNode as HTMLDivElement;
  arrowsContainer.childNodes.forEach((node) => {
    (node as HTMLDivElement).classList.add(popoverArrowClass);
  });

  // listen to the unseen count changed event propagated from the web component
  document.addEventListener("novu:unseen_count_changed", (event) => {
    unseenCount.value = (event as CustomEvent).detail as number;
  });
});

const calculateStyles = ({
  styles,
  theme: propsTheme,
  colorScheme,
}: Pick<
  NotificationCenterComponentProps,
  "theme" | "styles" | "colorScheme"
>) => {
  const { theme, common } = getDefaultTheme({
    colorScheme,
    theme: propsTheme,
  });
  const { bellColors } = getDefaultBellColors({
    colorScheme,
    bellColors: {},
  });

  return {
    popoverArrowClass: css(
      getStyleByPath({
        styles,
        path: "popover.arrow",
        theme,
        common,
        colorScheme,
      })
    ),
    popoverDropdownClass: css(
      getStyleByPath({
        styles,
        path: "popover.dropdown",
        theme,
        common,
        colorScheme,
      })
    ),
    bellButtonClass: css(
      getStyleByPath({
        styles,
        path: "bellButton.root",
        theme,
        common,
        colorScheme,
      })
    ),
    gradientDotClass: css(
      getStyleByPath({
        styles,
        path: "bellButton.dot",
        theme,
        common,
        colorScheme,
      })
    ),
    bellColors,
  };
};

const computedStyles = computed(() => {
  const {
    popoverArrowClass,
    popoverDropdownClass,
    bellButtonClass,
    gradientDotClass,
    bellColors,
  } = calculateStyles({
    theme: props.theme,
    styles: props.styles,
    colorScheme: props.colorScheme,
  });
  return {
    popoverArrowClass,
    popoverDropdownClass,
    bellButtonClass,
    gradientDotClass,
    bellColors,
  };
});

const {
  popoverArrowClass,
  popoverDropdownClass,
  bellButtonClass,
  gradientDotClass,
  bellColors,
} = computedStyles.value;
</script>

<template>
  <VDropdown
    :theme="colorScheme"
    :popperClass="popoverDropdownClass"
    :placement="popover?.position"
    :distance="popover?.offset"
  >
    <!-- Popover target - usually button -->
    <slot v-if="hasSlot" v-bind="unseenCount" :unseen-count="unseenCount" />
    <BellButton
      v-else
      :bellButtonClass="bellButtonClass"
      :gradientDotClass="gradientDotClass"
      :unseenCount="unseenCount"
      :colors="bellColors"
    />

    <!-- Popover content -->
    <template #popper>
      <notification-center-content-component
        ref="popper"
        :backendUrl="backendUrl"
        :socketUrl="socketUrl"
        :subscriberId="subscriberId"
        :applicationIdentifier="applicationIdentifier"
        :subscriberHash="subscriberHash"
        :stores="stores"
        :tabs="tabs"
        :showUserPreferences="showUserPreferences"
        :theme="theme"
        :styles="styles"
        :colorScheme="colorScheme"
        :i18n="i18n"
        :sessionLoaded="sessionLoaded"
        :notificationClicked="notificationClicked"
        :unseenCountChanged="unseenCountChanged"
        :actionClicked="actionClicked"
        :tabClicked="tabClicked"
      />
    </template>
  </VDropdown>
</template>

<style scoped>
.v-popper--theme-light .v-popper__inner,
.v-popper--theme-dark .v-popper__inner {
  background: transparent;
  border: none;
  box-shadow: none;
}

.v-popper--theme-light .v-popper__arrow-outer,
.v-popper--theme-light .v-popper__arrow-inner {
  border-color: #fff;
}

.v-popper--theme-dark .v-popper__arrow-outer,
.v-popper--theme-dark .v-popper__arrow-inner {
  border-color: #1e1e26;
}
</style>
