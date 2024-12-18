<template>
    <footer class="footer">
      <div class="footer-content">
        <div class="footer-section">
          <h3>SUIVEZ-NOUS</h3>
          <div class="social-icons">
            <a href="#" class="social-icon"><i class="fab fa-facebook-f"></i></a>
            <a href="#" class="social-icon"><i class="fab fa-instagram"></i></a>
            <a href="#" class="social-icon"><i class="fab fa-pinterest"></i></a>
          </div>
        </div>
        <div class="footer-section">
          <h3>NOUS CONTACTER</h3>
          <p>xmastore@hohoho.com</p>
          <p class="address">25 Avenue des Sapins, 75008 Paris</p>
          <p>HO HO HO XMASTORE</p>
          <p>+32 4 23 45 67 89</p>
        </div>
        <div class="footer-section">
          <h3>NEWSLETTER</h3>
          <p>Recevez en avant-première nos offres de Noël et nos nouveautés !</p>
          <div class="email-input">
            <input type="email" placeholder="Votre email" v-model="email" />
            <button @click="subscribe">
              <i class="fas fa-paper-plane"></i>
            </button>
          </div>
        </div>
        <div class="footer-section logo-section">
          <img 
            src="@/assets/images/logo/Logo2.jpg" 
            alt="logo hohoho" 
            class="footer-logo"
            @click="scrollToTop"
          />
        </div>
      </div>
      <div class="copyright" @click="scrollToTop">© 2024 HOHOHO Xmastore. Tous droits réservés</div>
    </footer>
  </template>
  
  <script>
  export default {
    name: 'TheFooter',
    data() {
      return {
        email: ''
      }
    },
    methods: {
      subscribe() {
        if (this.validateEmail(this.email)) {
          alert('Merci pour votre inscription !')
          this.email = ''
        } else {
          alert('Veuillez entrer une adresse email valide.')
        }
      },
      scrollToTop() {
        window.scrollTo({
          top: 0,
          behavior: 'smooth'
        })
      },
      validateEmail(email) {
        const re = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
        return re.test(String(email).toLowerCase())
      }
    }
  }
  </script>
  
  <style scoped>
  .footer {
    background-color: #0B3B24;
    color: #fff;
    padding: 40px 0 20px;
    font-size: 14px;
  }
  
  .footer-content {
    display: flex;
    justify-content: space-between;
    max-width: 1400px;
    margin: 0 auto;
    padding: 0 20px;
  }
  
  .footer-section {
    flex: 1;
    margin-right: 20px;
  }
  
  h3 {
    color: #fff;
    margin-bottom: 15px;
    font-size: 16px;
    transition: color 0.3s ease;
  }
  
  h3:hover {
    color: #CC0000;
  }
  
  .social-icons {
    display: flex;
    gap: 15px;
  }
  
  .social-icon {
    color: #fff;
    font-size: 20px;
    transition: color 0.3s ease, transform 0.3s ease;
  }
  
  .social-icon:hover {
    color: #CC0000;
    transform: translateY(-3px);
  }
  
  .email-input {
    display: flex;
    margin-top: 10px;
  }
  
  .email-input input {
    flex-grow: 1;
    padding: 8px;
    border: none;
    border-radius: 4px 0 0 4px;
  }
  
  .email-input button {
    background-color: #CC0000;
    color: white;
    border: none;
    padding: 8px 15px;
    border-radius: 0 4px 4px 0;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }
  
  .email-input button:hover {
    background-color: #8B0000;
  }
  
  .logo-section {
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  .footer-logo {
    width: 100px;
    height: auto;
    transition: transform 0.3s ease;
    cursor: pointer;
  }
  
  .footer-logo:hover {
    transform: scale(1.1);
  }
  
  .copyright {
    text-align: center;
    color: #fff;
    padding: 20px;
    cursor: pointer;
    transition: color 0.3s ease;
  }
  
  .copyright:hover {
    color: #CC0000;
  }
  
  @media (max-width: 768px) {
    .footer-content {
      flex-direction: column;
      gap: 30px;
    }
  
    .footer-section {
      margin-right: 0;
      text-align: center;
    }
  
    .social-icons {
      justify-content: center;
    }
    
    .email-input {
      max-width: 300px;
      margin: 10px auto;
    }
  }
  </style>
