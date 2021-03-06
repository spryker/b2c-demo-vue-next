<template>
  <div id="wishlist">
    <SfSidebar
      :visible="isWishlistSidebarOpen"
      :button="false"
      title="My Wishlist"
      @close="toggleWishlistSidebar"
      class="sidebar sf-sidebar--right"
    >
      <template #title>
        <div class="heading__wrapper">
          <SfHeading :level="3" title="My wishlist" class="sf-heading--left" />
          <SfButton
            data-cy="wishlist-sidebar-button_toggle-wishlist"
            class="heading__close-button sf-button--pure"
            aria-label="Wishlist sidebar close button"
            @click="toggleWishlistSidebar"
          >
            <SfIcon icon="cross" size="14px" color="gray-primary" />
          </SfButton>
        </div>
      </template>
      <transition name="fade" mode="out-in">
        <div v-if="totalItems" class="my-wishlist" key="my-wishlist">
          <div class="my-wishlist__total-items">
            Total items: <strong>{{ totalItems }}</strong>
          </div>
          <div class="collected-product-list">
            <transition-group name="fade" tag="div">
              <SfCollectedProduct
                data-cy="collected-product-wishlist-sidebar"
                v-for="product in products"
                :key="wishlistGetters.getItemSku(product)"
                :image="wishlistGetters.getItemImage(product)"
                :title="wishlistGetters.getItemName(product)"
                :regular-price="
                  wishlistGetters.getFormattedPrice(
                    wishlistGetters.getItemPrice(product).regular,
                  )
                "
                :special-price="
                  wishlistGetters.getFormattedPrice(
                    wishlistGetters.getItemPrice(product).special,
                  )
                "
                :stock="99999"
                image-width="180"
                image-height="200"
                @click:remove="removeItem({ product })"
                class="collected-product"
              >
                <template #configuration>
                  <div class="collected-product__properties">
                    <SfProperty
                      v-for="attribute in wishlistGetters.getItemAttributes(
                        product,
                      )"
                      :key="attribute.value"
                      :name="attribute.label"
                      :value="attribute.value"
                    />
                  </div>
                </template>
                <template #input="{}">&nbsp;</template>
                <template #actions>
                  <SfButton
                    class="sf-button--text desktop-only"
                    @click="addItem({ product, quantity: 1 })"
                  >
                    Add to the cart
                  </SfButton>
                </template>
              </SfCollectedProduct>
            </transition-group>
          </div>
          <div class="sidebar-bottom">
            <SfProperty
              class="sf-property--full-width my-wishlist__total-price"
            >
              <template #name>
                <span class="my-wishlist__total-price-label">Total price:</span>
              </template>
              <template #value>
                <SfPrice
                  :regular="wishlistGetters.getFormattedPrice(totals.subtotal)"
                />
              </template>
            </SfProperty>
          </div>
        </div>
        <div v-else class="empty-wishlist" key="empty-wishlist">
          <div class="empty-wishlist__banner">
            <img
              src="@storefront-ui/shared/icons/empty_cart.svg"
              alt
              class="empty-wishlist__icon"
            />
            <h3 class="empty-wishlist__label">Your bag is empty</h3>
            <p class="empty-wishlist__description">
              Looks like you haven’t added any items to the bag yet. Start
              shopping to fill it in.
            </p>
          </div>
          <SfButton
            data-cy="wishlist-sidebar-btn_start-shopping"
            @click="toggleWishlistSidebar"
            class="sf-button--full-width color-secondary"
            >Start shopping</SfButton
          >
        </div>
      </transition>
    </SfSidebar>
  </div>
</template>
<script>
import {
  SfSidebar,
  SfHeading,
  SfButton,
  SfIcon,
  SfProperty,
  SfPrice,
  SfCollectedProduct,
} from '@storefront-ui/vue';
import { computed } from '@vue/composition-api';
import {
  useWishlist,
  useUser,
  useCart,
  wishlistGetters,
} from '@spryker-vsf/composables';
import { onSSR } from '@vue-storefront/core';
import { useUiState } from '~/composables';

export default {
  name: 'Wishlist',
  components: {
    SfSidebar,
    SfButton,
    SfHeading,
    SfIcon,
    SfProperty,
    SfPrice,
    SfCollectedProduct,
  },
  setup() {
    const { isWishlistSidebarOpen, toggleWishlistSidebar } = useUiState();
    const { wishlist, removeItem } = useWishlist();
    const { isAuthenticated } = useUser();
    const { addItem } = useCart();
    const products = computed(() => wishlistGetters.getItems(wishlist.value));
    const totals = computed(() => wishlistGetters.getTotals(wishlist.value));
    const totalItems = computed(() =>
      wishlistGetters.getTotalItems(wishlist.value),
    );

    return {
      isAuthenticated,
      products,
      removeItem,
      isWishlistSidebarOpen,
      toggleWishlistSidebar,
      totals,
      totalItems,
      wishlistGetters,
      addItem,
    };
  },
};
</script>

<style lang="scss" scoped>
.sidebar {
  --sidebar-top-padding: var(--spacer-lg) var(--spacer-base) 0
    var(--spacer-base);
  --sidebar-content-padding: var(--spacer-lg) var(--spacer-base);
}

.my-wishlist {
  flex: 1;
  display: flex;
  flex-direction: column;
  &__total-items {
    font: var(--font-weight--normal) var(--font-size--lg) / 1.6
      var(--font-family--secondary);
    color: var(--c-link);
    margin: 0;
  }
  &__total-price {
    --property-name-font-size: var(--font-size--xl);
    --price-font-size: var(--font-size--xl);
    margin: 0 0 var(--spacer-xl) 0;
    &-label {
      font: var(--font-weight--normal) var(--font-size--2xl) / 1.6
        var(--font-family--secondary);
      color: var(--c-link);
    }
  }
}
.empty-wishlist {
  display: flex;
  flex: 1;
  flex-direction: column;
  &__banner {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
  }
  &__label,
  &__description {
    text-align: center;
  }
  &__label {
    margin: var(--spacer-2xl) 0 0 0;
    font: var(--font-weight--normal) var(--font-size--xl) / 1.6
      var(--font-family--secondary);
    color: var(--c-primary);
  }
  &__description {
    margin: var(--spacer-sm) 0 0 0;
    font: var(--font-weight--normal) var(--font-size--base) / 1.6
      var(--font-family--primary);
    color: var(--c-link);
  }
  &__icon {
    width: 18.125rem;
    height: 12.3125rem;
    margin-left: 50%;
    @include for-desktop {
      margin-left: 45%;
    }
  }
}
.heading {
  &__wrapper {
    --heading-title-color: var(--c-link);
    --heading-title-font-weight: var(--font-weight--semibold);
    display: flex;
    justify-content: space-between;
  }
}

.sidebar-bottom {
  margin: auto 0 0 0;
}

.collected-product {
  margin: var(--spacer-base) 0;
  &__properties {
    margin: var(--spacer-sm) 0 0 0;
  }
  ::v-deep .sf-collected-product__remove--circle-icon {
    --button-background: var(--c-primary);
    --icon-color: var(--c-white);
  }
}
</style>
