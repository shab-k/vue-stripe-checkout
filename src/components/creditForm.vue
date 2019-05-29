<template>
<div id="app" v-cloak>
   <div class="uk-align-center uk-margin uk-width-large uk-background-muted uk-box-shadow-large">
    <form @submit="checkForm()" class="uk-padding" action="" method="post">
      <h2>Your Payment Details</h2>
      <div class="uk-margin uk-text-center">
          <p class="stripeError" v-if="stripeError">{{ stripeError }}</p>
      </div> 
        <!-- Name -->
        <!-- Name didn't work with stripe elements -->
        <!-- Couldn't use input tag since I received an error related to Stripe, so I couldn't use materialize CSS -->
         <div class="uk-margin uk-text-left">
          <p v-if="errors.length">
            <ul>
              <li v-for="(error, index) in errors" v-bind:key="index">{{ error }}</li>
            </ul>
          </p>
            <label class="uk-form-label" for="Name"><i class="far fa-user"></i> Name</label>
            <div class="uk-form-controls">
              <input type="text" required id="card-name" v-model="name" name="name" class="field uk-input" placeholder="John Doe" :class="{ 'uk-form-danger': cardNameError }">
              <span class="help-block" v-if="cardNameError">{{cardNameError}}</span>
            </div>
          </div> 
          <!-- Card -->
         <div class="uk-width-2-3@s uk-margin uk-text-left">
            <label class="uk-form-label" for="Card Number"> <i class="fas fa-credit-card"></i> Card Number</label>
            <div class="uk-form-controls">
              
              <div id="card-number" class="uk-input" :class="{ 'uk-form-danger': cardNumberError }"></div>
              <span class="help-block" v-if="cardNumberError">{{cardNumberError}}</span>
            </div>
          </div> 
          <div class="uk-grid-small uk-text-left" uk-grid>
            <!-- CVC -->
            <div class="uk-width-1-2@s">
              <label class="uk-form-label" for="Card CVC"><span uk-tooltip="3-digit security code usually found on the back of your card. American Express cards have a 4-digit code located on the front." class="tooltiptext"><i class="far fa-question-circle"></i> CVC</span></label>
              <div class="uk-form-controls">
                <div id="card-cvc" class="uk-input" :class="{ 'uk-form-danger': cardCvcError }"></div>
                <span class="help-block" v-if="cardCvcError">{{cardCvcError}}</span>
              </div>
            </div>
            <!-- Expiry -->
            <div class="uk-width-1-2@s">
              <label class="uk-form-label" for="Expiry Month"><i class="fas fa-calendar-times"></i> Expiry</label>
              <div class="uk-form-controls">
                <div id="card-expiry" class="uk-input" :class="{ 'uk-form-danger': cardExpiryError }"></div>
                <span class="help-block" v-if="cardExpiryError">{{cardExpiryError}}</span>
              </div>
            </div>
          </div> 
          <!-- Buttons -->
        <div class="uk-margin uk-margin-remove-bottom">
            <button type="reset" class="uk-button uk-button-small uk-button-default" @click.prevent="reset()">reset</button>
            <button type="submit" class="uk-button uk-button-small uk-button-primary" @click.prevent="submitFormToCreateToken()">Pay $25</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  name: 'creditForm',
  data() {
    return{
      name: null,
      errors: [],
      card: {
      cvc: '',
      number: '',
      expiry: ''
    },

    //elements
    cardName: '',
    cardNumber: '',
    cardExpiry: '',
    cardCvc: '',
    stripe: null,

    // errors
    stripeError: '',
    cardCvcError: '',
    cardExpiryError: '',
    cardNumberError: '',
    cardNameError: '',
    loading: false
     }

  },

  mounted() {
    this.setUpStripe();
  },

  methods: {
    checkForm(e) {
      if (this.name) {
        return true;
      }
      this.errors = [];
      if (this.name === '') {
        this.errors.push("Name required!");
      }
      e.preventDefault();
    },
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
          // this.cardName = elements.create('cardName');

          this.cardCvc.mount('#card-cvc');
          this.cardExpiry.mount('#card-expiry');
          this.cardNumber.mount('#card-number');
          // this.cardName.mount('#card-name');

          this.listenForErrors();
        }
      },

      listenForErrors() {
        const vm = this;

        // this.cardName.addEventListener('change', (event) => {
        //   vm.toggleError(event);
        //   vm.cardNameError = ''
        //   vm.card.name = event.complete ? true : false
        // });

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

        if (!this.card.name) {
          valid = false;
          this.cardNameError = "Name is Required";
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
              const token = result.token.id;
              alert('Payment Received! Thank You!')
                //send the token to the server
                //clear the inputs
            }
          })
      },
     
      clearElementsInputs() {
        this.cardCvc.clear()
        this.cardExpiry.clear()
        this.cardNumber.clear()
        this.cardName.clear()
      },

      clearCardErrors() {
        this.stripeError = ''
        this.cardCvcError = ''
        this.cardExpiryError = ''
        this.cardNumberError = ''
        this.cardNameError = ''
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
</style>
