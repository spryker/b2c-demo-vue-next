<template>
  <div id="checkout">
    <div class="checkout">
      <div class="checkout__main">
        <SfSteps
          :active="currentStep"
          v-if="currentStep < 4"
          class="checkout__steps"
        >
          <SfStep
            v-for="(step, index) in STEPS"
            :key="step.name"
            :name="step.label"
          >
            <nuxt-child
              @showReview="handleShowReview"
              @changeStep="updateStep($event)"
              @nextStep="handleNextStep(index + 1)"
            />
          </SfStep>
        </SfSteps>
        <nuxt-child v-else @changeStep="updateStep($event)" />
      </div>
      <div class="checkout__aside desktop-only" v-if="!isThankYou">
        <transition name="fade">
          <CartPreview v-if="showCartPreview" key="order-summary" />
          <OrderReview v-else key="order-review" />
        </transition>
      </div>
    </div>
  </div>
</template>
<script>
import { SfSteps, SfButton } from '@storefront-ui/vue';
import CartPreview from '~/components/Checkout/CartPreview';
import OrderReview from '~/components/Checkout/OrderReview';
import { ref, computed } from '@vue/composition-api';

const STEPS = [
  { name: 'personal-details', label: 'Personal Details' },
  { name: 'shipping', label: 'Shipping' },
  { name: 'payment', label: 'Payment' },
  { name: 'order-review', label: 'Review' },
];

export default {
  name: 'Checkout',
  components: {
    SfButton,
    SfSteps,
    CartPreview,
    OrderReview,
  },
  setup(props, context) {
    const showCartPreview = ref(true);
    const currentStep = ref(0);

    const handleShowReview = () => {
      showCartPreview.value = false;
    };

    const isThankYou = computed(() => currentStep.value === "thank-you");

    const updateStep = next => {
      currentStep.value = next;
    };

    const handleNextStep = nextStep => {
      context.root.$router.push(
        nextStep < 4 ? STEPS[nextStep].name : 'thank-you',
      );
    };

    return {
      STEPS,
      handleNextStep,
      isThankYou,
      currentStep,
      updateStep,
      handleShowReview,
      showCartPreview,
    };
  },
};
</script>

<style lang="scss" scoped>
#checkout {
  box-sizing: border-box;
  @include for-desktop {
    max-width: 1240px;
    margin: 0 auto;
  }
}
.checkout {
  @include for-desktop {
    display: flex;
  }
  &__main {
    @include for-desktop {
      flex: 1;
      padding: var(--spacer-xl) 0 0 0;
    }
  }
  &__aside {
    @include for-desktop {
      flex: 0 0 25.5rem;
      margin: 0 0 0 4.25rem;
    }
  }
  &__steps {
    --steps-content-padding: 0 var(--spacer-base);
    @include for-desktop {
      --steps-content-padding: 0;
    }

    &-auth::v-deep .sf-steps__step:first-child {
      --steps-step-color: #e8e4e4;
    }
  }
}
</style>
