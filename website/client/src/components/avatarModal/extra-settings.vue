<template>
  <div
    id="extra"
    class="customize-section d-flex flex-column"
    :class="{ 'justify-content-between': !showEmptySection}"
  >
    <sub-menu
      class="text-center"
      :items="extraSubMenuItems"
      :active-sub-page="activeSubPage"
      @changeSubPage="changeSubPage($event)"
    />
    <div
      v-if="activeSubPage === 'glasses'"
      id="hair-color"
    >
      <customize-options :items="eyewear" />
    </div>
    <div
      v-if="activeSubPage === 'ears'"
      id="animal-ears"
    >
      <customize-options
        v-if="animalItems('back').length > 0"
        :items="animalItems('headAccessory')"
      />
    </div>
    <div
      v-if="activeSubPage === 'tails'"
      id="animal-tails"
    >
      <customize-options
        v-if="animalItems('back').length > 0"
        :items="animalItems('back')"
      />
    </div>
    <div
      v-if="activeSubPage === 'headband'"
      id="headband"
    >
      <customize-options :items="headbands" />
    </div>
    <div
      v-if="activeSubPage === 'wheelchair'"
      id="wheelchairs"
    >
      <customize-options :items="chairs" />
    </div>
    <div
      v-if="activeSubPage === 'flower'"
      id="flowers"
    >
      <customize-options :items="flowers" />
    </div>
    <div
      v-if="showEmptySection"
      class="my-5"
    >
      <h3
        v-once
      >
        {{ $t('noItemsOwned') }}
      </h3>
      <p
        v-once
        class="w-50 mx-auto"
        v-html="$t('visitCustomizationsShop')"
      ></p>
    </div>
    <customize-banner
      v-else-if="editing"
    />
  </div>
</template>

<script>
import appearance from '@/../../common/script/content/appearance';
import { subPageMixin } from '../../mixins/subPage';
import { userStateMixin } from '../../mixins/userState';
import { avatarEditorUtilities } from '../../mixins/avatarEditUtilities';
import customizeBanner from './customize-banner';
import customizeOptions from './customize-options';
import subMenu from './sub-menu';

const freeShirtKeys = Object.keys(appearance.shirt).filter(k => appearance.shirt[k].price === 0);
const specialShirtKeys = Object.keys(appearance.shirt).filter(k => appearance.shirt[k].price !== 0);

export default {
  components: {
    customizeBanner,
    customizeOptions,
    subMenu,
  },
  mixins: [
    subPageMixin,
    userStateMixin,
    avatarEditorUtilities,
  ],
  props: [
    'editing',
  ],
  data () {
    return {
      animalItemKeys: {
        back: ['bearTail', 'cactusTail', 'foxTail', 'lionTail', 'pandaTail', 'pigTail', 'tigerTail', 'wolfTail'],
        headAccessory: ['bearEars', 'cactusEars', 'foxEars', 'lionEars', 'pandaEars', 'pigEars', 'tigerEars', 'wolfEars'],
      },
      chairKeys: ['none', 'black', 'blue', 'green', 'pink', 'red', 'yellow', 'handleless_black', 'handleless_blue', 'handleless_green', 'handleless_pink', 'handleless_red', 'handleless_yellow'],
      specialShirtKeys,
      items: [
        {
          id: 'size',
          label: this.$t('size'),
        },
        {
          id: 'shirt',
          label: this.$t('shirt'),
        },
      ],
    };
  },
  computed: {
    extraSubMenuItems () {
      const items = [];

      if (this.editing) {
        items.push({
          id: 'glasses',
          label: this.$t('glasses'),
        });
      }

      items.push(
        {
          id: 'wheelchair',
          label: this.$t('wheelchair'),
        },
        {
          id: 'flower',
          label: this.$t('accent'),
        },
      );

      if (this.editing) {
        items.push({
          id: 'ears',
          label: this.$t('animalEars'),
        });

        items.push({
          id: 'tails',
          label: this.$t('animalTails'),
        });

        items.push({
          id: 'headband',
          label: this.$t('headband'),
        });
      }

      return items;
    },
    eyewear () {
      const keys = [
        'blackTopFrame', 'blueTopFrame', 'greenTopFrame', 'pinkTopFrame', 'redTopFrame', 'whiteTopFrame', 'yellowTopFrame',
        'blackHalfMoon', 'blueHalfMoon', 'greenHalfMoon', 'pinkHalfMoon', 'redHalfMoon', 'whiteHalfMoon', 'yellowHalfMoon',
      ];
      const noneOption = this.createGearItem(0, 'eyewear', 'base');
      noneOption.none = true;
      const options = [
        noneOption,
      ];

      for (const key of keys) {
        const newKey = `eyewear_special_${key}`;
        const option = {};
        option.key = key;
        option.active = this.user.preferences.costume
          ? this.user.items.gear.costume.eyewear === newKey
          : this.user.items.gear.equipped.eyewear === newKey;
        option.class = `eyewear_special_${key}`;
        option.click = () => {
          const type = this.user.preferences.costume ? 'costume' : 'equipped';

          return this.equip(newKey, type);
        };
        options.push(option);
      }

      return options;
    },
    freeShirts () {
      return freeShirtKeys.map(s => this.mapKeysToFreeOption(s, 'shirt'));
    },
    specialShirts () {
      let backgroundUpdate = this.backgroundUpdate; // eslint-disable-line
      const keys = this.specialShirtKeys;
      const options = keys.map(key => this.mapKeysToOption(key, 'shirt'));
      return options;
    },
    headbands () {
      const keys = ['blackHeadband', 'blueHeadband', 'greenHeadband', 'pinkHeadband', 'redHeadband', 'whiteHeadband', 'yellowHeadband'];
      const noneOption = this.createGearItem(0, 'headAccessory', 'base', 'headband');
      noneOption.none = true;
      const options = [
        noneOption,
      ];

      for (const key of keys) {
        const option = this.createGearItem(key, 'headAccessory', 'special', 'headband');
        const newKey = `headAccessory_special_${key}`;
        option.click = () => {
          const type = this.user.preferences.costume ? 'costume' : 'equipped';
          return this.equip(newKey, type);
        };

        options.push(option);
      }

      return options;
    },
    chairs () {
      const options = this.chairKeys.map(key => {
        const option = {};
        option.key = key;
        if (key === 'none') {
          option.none = true;
        }
        option.active = this.user.preferences.chair === key;
        option.class = `button_chair_${key} chair ${key.includes('handleless_') ? 'handleless' : ''}`;
        option.click = () => this.set({ 'preferences.chair': key });
        return option;
      });
      return options;
    },
    flowers () {
      const keys = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16];
      const options = keys.map(key => {
        const option = {};
        option.key = key;
        if (key === 0) {
          option.none = true;
        }
        option.active = this.user.preferences.hair.flower === key;
        option.class = `icon_hair_flower_${key} flower`;
        option.click = () => this.set({ 'preferences.hair.flower': key });
        return option;
      });
      return options;
    },
    showEmptySection () {
      switch (this.activeSubPage) {
        case 'ears':
          return this.editing && this.animalItems('headAccessory').length === 1;
        case 'tails':
          return this.editing && this.animalItems('back').length === 1;
        default:
          return false;
      }
    },
  },
  mounted () {
    this.changeSubPage(this.extraSubMenuItems[0].id);
  },
  methods: {
    animalItems (category) {
      // @TODO: For some resonse when I use $set on the
      // user purchases object, this is not recomputed. Hack for now
      let backgroundUpdate = this.backgroundUpdate; // eslint-disable-line
      const keys = this.animalItemKeys[category];

      const noneOption = this.createGearItem(0, category, 'base', category);
      noneOption.none = true;
      const options = [
        noneOption,
      ];

      for (const key of keys) {
        const newKey = `${category}_special_${key}`;
        const userPurchased = this.user.items.gear.owned[newKey];
        if (userPurchased) {
          const option = {};
          option.key = key;
          option.active = this.user.preferences.costume
            ? this.user.items.gear.costume[category] === newKey
            : this.user.items.gear.equipped[category] === newKey;
          option.class = `headAccessory_special_${option.key} ${category}`;
          if (category === 'back') {
            option.class = `icon_back_special_${option.key} back`;
          }
          option.click = () => {
            const type = this.user.preferences.costume ? 'costume' : 'equipped';
            return this.equip(newKey, type);
          };
          options.push(option);
        }
      }

      return options;
    },
    animalItemsUnlockString (category) {
      const keys = this.animalItemKeys[category].map(key => `items.gear.owned.${category}_special_${key}`);

      return keys.join(',');
    },
    createGearItem (key, gearType, subGearType, additionalClass) {
      const newKey = `${gearType}_${subGearType ? `${subGearType}_` : ''}${key}`;
      const option = {};
      option.key = key;
      const visibleGearType = this.user.preferences.costume ? 'costume' : 'equipped';
      const currentlyEquippedValue = this.user.items.gear[visibleGearType][gearType];

      option.active = currentlyEquippedValue === newKey;

      if (key === 0) {
        // if key is the "none" option check if a property
        // doesn't have a value and mark it as active
        option.active = option.active || !currentlyEquippedValue;
      }

      option.class = `${newKey} ${additionalClass}`;
      option.click = () => {
        const type = this.user.preferences.costume ? 'costume' : 'equipped';
        const currentlyEquipped = this.user.items.gear[type][gearType];

        // no need to call api/equip-op if its already selected
        if (currentlyEquipped === newKey || (key === 0 && !currentlyEquipped)) {
          return;
        }

        let keyToEquip = newKey;

        if (option.none) {
          // you need to "equip" the current selected AGAIN in order to un-equip it
          // the "none-key" isn't allowed to be sent
          keyToEquip = currentlyEquipped;
        }

        this.equip(keyToEquip, type);
      };

      return option;
    },
  },
};
</script>
