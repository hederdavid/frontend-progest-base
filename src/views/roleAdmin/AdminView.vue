<template>
  <div class="admin-layout">
    <!-- Sidebar fixo -->
    <Sidebar class="sidebar" />
    <!-- Conteúdo principal -->
    <main class="content">
      <Header :userName="userName" :userRole="userRole" :headerName="headerName" />
      <section class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4 p-6 h-2/3 text-off-white">
        <Card v-for="(card, index) in cards" :key="index" :title="card.title" :number="card.number" :icon="card.icon"
          :cardClass="card.cardClass" :buttonClass="card.buttonClass" />
      </section>
    </main>
  </div>
</template>

<script>
import Sidebar from "@/components/roleAdmin/Sidebar.vue";
import Header from "@/components/roleAdmin/Header.vue";
import Card from "@/components/roleAdmin/Card.vue";
import axios from "axios";
import { API_URL } from '@/config';

export default {
  components: {
    Sidebar,
    Header,
    Card,
  },
  data() {
    return {
      apiUrl: API_URL,
      userName: "",
      userRole: "Admin",
      headerName: '',
      cards: [
        {
          title: "Pedidos pendentes",
          icon: "../../src/components/icons/carrinho.svg",
          cardClass: "bg-yellow-300",
          buttonClass: "bg-yellow-400 hover:bg-yellow-500",
        },
        {
          title: "Usuários",
          icon: "../../src/components/icons/adicionar-usuario.svg",
          cardClass: "bg-blue-300",
          buttonClass: "bg-blue-400 hover:bg-blue-500",
        },
        {
          title: "Produtos abaixo do mínimo",
          icon: "../../src/components/icons/sirene.svg",
          cardClass: "bg-red-300",
          buttonClass: "bg-red-400 hover:bg-red-500",
        },
        {
          title: "Produtos próximo ao vencimento",
          icon: "../../src/components/icons/atencao.svg",
          cardClass: "bg-orange-300",
          buttonClass: "bg-orange-400 hover:bg-orange-500",
        },
        {
          title: "Farmácias",
          icon: "../../src/components/icons/farmacia.svg",
          cardClass: "bg-green-300",
          buttonClass: "bg-green-400 hover:bg-green-500",
        },
      ],
    };
  },
  created() {
    this.fetchUserInfo();
    this.fetchData();
  },
  methods: {
    async fetchUserInfo() {
      try {
        // Recupera o token do localStorage
        const token = localStorage.getItem("token");
        if (!token) {
          console.error("Token não encontrado");
          this.$router.push("/login");
          return;
        }

        // Define os headers com o token
        const headers = {
          Authorization: `Bearer ${token}`,
        };

        // Faz a requisição para obter informações do usuário
        const response = await axios.get(`${this.apiUrl}/user`, {
          headers,
        });

        // Atualiza os dados do usuário
        this.userName = response.data.name; // Certifique-se de que o nome vem no campo correto
        this.userRole = response.data.role;
      } catch (error) {
        console.error("Erro ao buscar informações do usuário:", error);

        // Redireciona para login em caso de erro (ex.: token expirado ou inválido)
        this.$router.push("/login");
      }
    },
    async fetchData() {
      try {
        const token = localStorage.getItem("token");
        const headers = {
          Authorization: `Bearer ${token}`,
        };

        const responseCountUsers = await axios.get(
          `${this.apiUrl}/countUsers`,
          { headers }
        );

        const responseBelowMinimumStock = await axios.get(
          `${this.apiUrl}/count-below-minimum-stock`,
          { headers }
        );
        const responseExpiringSoon = await axios.get(
          `${this.apiUrl}/count-expiring-soon`,
          { headers }
        );

        this.cards[0].number = 0;
        this.cards[1].number = responseCountUsers.data.count;
        this.cards[2].number = responseBelowMinimumStock.data.count;
        this.cards[3].number = responseExpiringSoon.data.count;
        this.cards[4].number = 7;
      } catch (error) {
        console.error(error);
      }
    },
  },
};
</script>

<style lang="scss">
.admin-layout {
  display: flex;
  height: 100vh;

  .content {
    flex: 1;
    /* Faz o conteúdo ocupar o restante do espaço */
    overflow-y: auto;
    background-color: var(--light);
  }
}

:root {
  --primary: #bbbfff;
  --grey: #64748b;
  --dark: rgba(0, 77, 153, 1);
  --dark-alt: #002c6b;
  --light: #f1f5f9;
  --sidebar-width: 300px;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background: var(--light);
}

button {
  cursor: pointer;
  appearance: none;
  border: none;
  outline: none;
  background: none;
}

.app {
  display: flex;

  main {
    flex: 1 1 0;
    padding: 2rem;

    @media (max-width: 768px) {
      padding-left: 6rem;

      .content {
        flex: 1;
        /* Faz o conteúdo ocupar o restante do espaço */
        overflow-x: auto;
        background-color: var(--light);
      }
    }
  }
}
</style>