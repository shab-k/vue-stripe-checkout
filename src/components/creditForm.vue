<template>
<div id="app" v-cloak>
  <div class="uk-align-center uk-margin-top uk-width-large uk-background-muted uk-box-shadow-large">
    <form class="uk-padding">
      <h2>Your Payment Details</h2>
      <div class="uk-margin uk-text-center">
          <p class="stripeError" v-if="stripeError">{{stripeError}}</p>
      </div>
          <!-- Name -->
          <div class="uk-margin uk-text-left">
            <label class="uk-form-label" for="Name">
              Name
            </label>
            <div class="uk-form-controls">
              <input id="card-Holder-Name" class="uk-input" :class="{ 'uk-form-danger': nameError }">
              <span class="help-block" v-if="nameError">{{nameError}}</span>
            </div>
          </div>
          <!-- Card -->
          <div class="uk-margin uk-text-left">
            <label class="uk-form-label" for="Card Number">
              Card Number
            </label>
            <div class="uk-form-controls">
              <div id="card-number" class="uk-input" :class="{ 'uk-form-danger': cardNumberError }"></div>
              <span class="help-block" v-if="cardNumberError">{{cardNumberError}}</span>
            </div>
          </div>

          <div class="uk-grid-small uk-text-left" uk-grid>
            <div class="uk-width-1-2@s">
              <label class="uk-form-label" for="Card CVC">
                Card CVC
              </label>
              <div class="uk-form-controls">
                <div id="card-cvc" class="uk-input" :class="{ 'uk-form-danger': cardCvcError }"></div>
                <span class="help-block" v-if="cardCvcError">{{cardCvcError}}</span>
              </div>
            </div>
            <div class="uk-width-1-2@s">
              <label class="uk-form-label" for="Expiry Month">Expiry</label>
              <div class="uk-form-controls">
                <div id="card-expiry" class="uk-input" :class="{ 'uk-form-danger': cardExpiryError }"></div>
                <span class="help-block" v-if="cardExpiryError">{{cardExpiryError}}</span>
              </div>
            </div>
          </div>

          <div class="uk-margin uk-margin-remove-bottom uk-text-right">
            <button class="uk-button uk-button-small uk-button-default" @click.prevent="reset()">
              reset
            </button>

            <button class="uk-button uk-button-small uk-button-primary" @click.prevent="submitFormToCreateToken()">
              Pay $25
            </button>
          </div>

        </form>
      </div>

    </div>
</template>

<script>
export default {
  name: 'creditForm',
  // props: {
  //   msg: String
  // },
// const app = new Vue({
//   el: '#app',
  data() {
    return{
      cardHolderName: '',
      card: {
      cvc: '',
      number: '',
      expiry: ''
    },

    //elements
    // name:'',
    cardNumber: '',
    cardExpiry: '',
    cardCvc: '',
    stripe: null,

    // errors
    stripeError: '',
    cardCvcError: '',
    cardExpiryError: '',
    cardNumberError: '',
    nameError: '',
    loading: false
     }

  },

  mounted() {
    this.setUpStripe();
  },

  methods: {
    setUpStripe() {
        if (window.Stripe === undefined) {
          alert('Stripe V3 library not loaded!');
        } else {
          const stripe = window.Stripe('pk_test_GZYfG6M52r52tEHYj4G5mUkz');
          this.stripe = stripe;

          const elements = stripe.elements();
          this.cardCvc = elements.create('cardCvc');
          this.cardExpiry = elements.create('cardExpiry');
          this.cardNumber = elements.create('cardNumber');

          this.cardCvc.mount('#card-cvc');
          this.cardExpiry.mount('#card-expiry');
          this.cardNumber.mount('#card-number');

          this.listenForErrors();
        }
      },

      listenForErrors() {
        const vm = this;

        this.cardHolderName.addEventListener('change', (event) => {
          vm.toggleError(event);
          vm.nameError = ''
          vm.cardHolderName = event.complete ? true : false
        });

        this.cardNumber.addEventListener('change', (event) => {
          vm.toggleError(event);
          vm.cardNumberError = ''
          vm.card.number = event.complete ? true : false
        });
				
        this.cardExpiry.addEventListener('change', (event) => {
          vm.toggleError(event);
          vm.cardExpiryError = ''
          vm.card.expiry = event.complete ? true : false
        });
        
				this.cardCvc.addEventListener('change', (event) => {
          vm.toggleError(event);
          vm.cardCvcError = ''
          vm.card.cvc = event.complete ? true : false
        });
      },

      toggleError (event) {
        if (event.error) {
          this.stripeError = event.error.message;
        } else {
          this.stripeError = '';
        }
      },

      submitFormToCreateToken() {
        this.clearCardErrors();
        let valid = true;

        if (!this.cardHolderName) {
          valid = false;
          this.nameError = "Name is Required";
        }
        if (!this.card.number) {
          valid = false;
          this.cardNumberError = "Card Number is Required";
        }
        if (!this.card.cvc) {
          valid = false;
          this.cardCvcError = "CVC is Required";
        }
        if (!this.card.expiry) {
          valid = false;
          this.cardExpiryError = "Month is Required";
        }
        if (this.stripeError) {
          valid = false;
        }
        if (valid) {
          this.createToken()
        }
      },

      createToken() {
        this.stripe.createToken(this.cardNumber).then((result) => {
            if (result.error) {
              this.stripeError = result.error.message;
            } else {
              const token = result.token.id
              alert('Thank You!')
                //send the token to your server
                //clear the inputs
            }
          })
      },

      clearElementsInputs() {
        this.cardCvc.clear()
        this.cardExpiry.clear()
        this.cardNumber.clear()
        this.cardHolderName.clear()
      },

      clearCardErrors() {
        this.stripeError = ''
        this.cardCvcError = ''
        this.cardExpiryError = ''
        this.cardNumberError = ''
        this.nameError = ''
      },
			
			reset() {
				this.clearElementsInputs()
				this.clearCardErrors()
			}
  }

}
</script>


<style>
  [v-cloak] {
  display: none;
}

.help-block {
  color: red;
  font-size: 13px;
}

form {
  border-width: 2px;
  border-style: solid;
  border-color: #4400ff;
}

label.uk-form-label {
  color: blue;
}

button.uk-button-primary {
  color: yellow;
  font-weight: bold;
  background-color: #1e87f0;
  margin: 0px 10px;
}

#card-number,
#card-cvc,
#card-expiry {
  padding-top: 10px;
}

.stripeError {
  color: red;
  font-style: italic;
}

</style>
