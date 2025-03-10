<script setup>
import { ref, onMounted, watch } from 'vue';

// Estado do formulário e lista de produtos
const showForm = ref(false);
const productName = ref('');
const productDescription = ref('');
const productImage = ref(null);
const previewImage = ref('');
const whatsappNumber = ref('');
const products = ref([]);

// Senha do vendedor (PIN de segurança)
const sellerPassword = '1234'; // Altere conforme necessário
const enteredPassword = ref('');

// Abrir e fechar o modal do formulário
const toggleForm = () => {
  showForm.value = !showForm.value;
};

// Atualiza a pré-visualização da imagem
const handleImageUpload = (event) => {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      previewImage.value = e.target.result;
    };
    reader.readAsDataURL(file);
    productImage.value = file;
  }
};

// Adicionar novo produto à lista
const submitForm = () => {
  if (!productName.value || !productDescription.value || !whatsappNumber.value || !previewImage.value) {
    alert('Preencha todos os campos!');
    return;
  }

  products.value.push({
    name: productName.value,
    description: productDescription.value,
    image: previewImage.value,
    whatsapp: whatsappNumber.value,
  });

  // Salvar no LocalStorage
  localStorage.setItem('products', JSON.stringify(products.value));

  // Limpar campos do formulário
  productName.value = '';
  productDescription.value = '';
  productImage.value = null;
  previewImage.value = '';
  whatsappNumber.value = '';

  // Fechar modal
  toggleForm();
};

// Abre o WhatsApp com o número informado
const openWhatsApp = (number) => {
  const url = `https://wa.me/${number.replace(/\D/g, '')}`;
  window.open(url, '_blank');
};

// Excluir produto (com senha de vendedor)
const deleteProduct = (index) => {
  const enteredPin = prompt('Digite a senha do vendedor para excluir este produto:');
  
  if (enteredPin === sellerPassword) {
    products.value.splice(index, 1);
    localStorage.setItem('products', JSON.stringify(products.value));
    alert('Produto excluído com sucesso!');
  } else {
    alert('Senha incorreta! Você não tem permissão para excluir este produto.');
  }
};

// Recuperar produtos salvos ao carregar a página
onMounted(() => {
  const storedProducts = localStorage.getItem('products');
  if (storedProducts) {
    products.value = JSON.parse(storedProducts);
  }
});

// Atualizar LocalStorage sempre que os produtos mudarem
watch(products, (newProducts) => {
  localStorage.setItem('products', JSON.stringify(newProducts));
}, { deep: true });

</script>

<template>
  <h1>Bem-vindo!</h1>

  <div class="top-bar">
    <button class="announce-btn" @click="toggleForm">
      Anunciar Produto
    </button>
  </div>

  <!-- Modal do formulário -->
  <div v-if="showForm" class="modal-overlay">
    <div class="modal">
      <h2>Informe os detalhes do produto</h2>
      <form @submit.prevent="submitForm">
        <label>Nome do Produto:</label>
        <input type="text" v-model="productName" required />

        <label>Descrição:</label>
        <textarea v-model="productDescription" required></textarea>

        <label>Imagem do Produto:</label>
        <input type="file" @change="handleImageUpload" accept="image/*" />
        <div v-if="previewImage" class="image-preview">
          <img :src="previewImage" alt="Pré-visualização" />
        </div>

        <label>Número do WhatsApp:</label>
        <input type="text" v-model="whatsappNumber" required placeholder="+55 99999-9999" />

        <div class="buttons">
          <button type="submit" class="submit-btn">Anunciar</button>
          <button type="button" class="close-btn" @click="toggleForm">Fechar</button>
        </div>
      </form>
    </div>
  </div>

  <!-- Exibir produtos cadastrados -->
  <div v-if="products.length" class="products-container">
    <h2>Produtos Anunciados</h2>
    <div class="products-grid">
      <div v-for="(product, index) in products" :key="index" class="product-card">
        <img :src="product.image" alt="Imagem do Produto" class="product-image" />
        <h3>{{ product.name }}</h3>
        <p>{{ product.description }}</p>
        <button class="whatsapp-btn" @click="openWhatsApp(product.whatsapp)">
          Contato no WhatsApp
        </button>
        <button class="delete-btn" @click="deleteProduct(index)">
          Excluir Produto
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Barra superior */
.top-bar {
  display: flex;
  justify-content: center;
  padding: 10px;
}

.announce-btn {
  background-color: #085120;
  color: white;
  border: none;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  border-radius: 5px;
}

.announce-btn:hover {
  background-color: #0056b3;
}

/* Estilo do Modal */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(33, 43, 29, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal {
  background: rgb(10, 132, 24);
  padding: 20px;
  border-radius: 8px;
  width: 400px;
}

form {
  display: flex;
  flex-direction: column;
}

label {
  font-weight: bold;
  margin-top: 10px;
}

input, textarea {
  width: 100%;
  padding: 8px;
  margin-top: 5px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

textarea {
  height: 80px;
}

.image-preview img {
  width: 100%;
  max-height: 150px;
  object-fit: cover;
  border-radius: 5px;
  margin-top: 10px;
}

.buttons {
  display: flex;
  justify-content: space-between;
  margin-top: 15px;
}

.submit-btn {
  background-color: #28a745;
  color: rgb(249, 245, 245);
  border: none;
  padding: 10px;
  cursor: pointer;
  border-radius: 5px;
}

.submit-btn:hover {
  background-color: #218838;
}

.close-btn {
  background-color: #dc3545;
  color: white;
  border: none;
  padding: 10px;
  cursor: pointer;
  border-radius: 5px;
}

.close-btn:hover {
  background-color: #e2dede;
}

/* Estilo dos produtos */
.products-container {
  margin: 20px auto;
  max-width: 800px;
}

.products-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  justify-content: center;
}

.product-card {
  background: rgb(10, 17, 39);
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0px 4px 6px rgba(183, 14, 14, 0.1);
  text-align: center;
  width: 250px;
}

.product-image {
  width: 100%;
  height: 150px;
  object-fit: cover;
  border-radius: 5px;
}

.whatsapp-btn, .delete-btn {
  padding: 10px;
  font-size: 14px;
  cursor: pointer;
  border-radius: 5px;
  margin-top: 10px;
  background-color: #1d5d1d;
}

.delete-btn {
  background-color: #ff4d4d;
  color: white;
}

.delete-btn:hover {
  background-color: #cc0000;
}
</style>