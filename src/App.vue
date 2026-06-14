<template>
  <div id="app" class="dashboard-container">
    <header class="dashboard-header">
      <h2>Product CRUD Dashboard</h2>
      <p class="dashboard-subtitle">Manage your inventory seamlessly</p>
    </header>

    <div class="card form-card">
      <h3>{{ isEditing ? 'Edit Product' : 'Add New Product' }}</h3>
      
      <div class="form-group">
        <div class="input-wrapper">
          <input 
            v-model="form.name" 
            placeholder="Product Name" 
            class="form-input"
          />
        </div>
        <div class="input-wrapper">
          <input 
            v-model.number="form.price" 
            type="number" 
            placeholder="Price" 
            class="form-input"
          />
        </div>
        
        <div class="form-actions">
          <button v-if="!isEditing" @click="createProduct" class="btn btn-primary">Add Product</button>
          <div v-else class="btn-group">
            <button @click="updateProduct" class="btn btn-success">Update</button>
            <button @click="cancelEdit" class="btn btn-secondary">Cancel</button>
          </div>
        </div>
      </div>
    </div>

    <div class="list-section">
      <h3>Product List <span class="badge">{{ products.length }}</span></h3>
      
      <transition-group name="list" tag="ul" class="product-list" v-if="products.length > 0">
        <li v-for="product in products" :key="product.id" class="product-item">
          <div class="product-info">
            <span class="product-name">{{ product.name }}</span>
            <span class="product-price">${{ product.price }}</span>
          </div>
          <div class="product-actions">
            <button @click="editProduct(product)" class="btn-icon btn-edit" title="Edit Product">
              Edit
            </button>
            <button @click="deleteProduct(product.id)" class="btn-icon btn-delete" title="Delete Product">
              Delete
            </button>
          </div>
        </li>
      </transition-group>
      
      <div v-else class="empty-state">
        <p>No products available. Add one above!</p>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

const API_URL = import.meta.env.VITE_API_URL;

export default {
  data() {
    return {
      products: [],
      form: { id: null, name: '', price: '' },
      isEditing: false
    };
  },
  mounted() {
    this.getProducts();
  },
  methods: {
    async getProducts() {
      try {
        const response = await axios.get(API_URL);
        this.products = response.data;
      } catch (error) {
        console.error("Error fetching products:", error);
      }
    },
    async createProduct() {
      if (!this.form.name || !this.form.price) return alert("Please fill in all fields");
      try {
        await axios.post(API_URL, { name: this.form.name, price: this.form.price });
        this.clearForm();
        this.getProducts();
      } catch (error) {
        console.error("Error creating product:", error);
      }
    },
    editProduct(product) {
      this.isEditing = true;
      this.form = { ...product };
    },
    async updateProduct() {
      try {
        await axios.put(`${API_URL}/${this.form.id}`, { name: this.form.name, price: this.form.price });
        this.clearForm();
        this.getProducts();
      } catch (error) {
        console.error("Error updating product:", error);
      }
    },
    async deleteProduct(id) {
      if (confirm("Are you sure you want to delete this product?")) {
        try {
          await axios.delete(`${API_URL}/${id}`);
          this.getProducts();
        } catch (error) {
          console.error("Error deleting product:", error);
        }
      }
    },
    clearForm() {
      this.form = { id: null, name: '', price: '' };
      this.isEditing = false;
    },
    cancelEdit() {
      this.clearForm();
    }
  }
};
</script>

<style scoped>
/* Modern CSS Reset & Variable Tokens */
.dashboard-container {
  --bg-main: #0f172a;
  --bg-card: #1e293b;
  --text-main: #f8fafc;
  --text-muted: #94a3b8;
  --primary: #3b82f6;
  --primary-hover: #2563eb;
  --success: #10b981;
  --success-hover: #059669;
  --danger: #ef4444;
  --danger-hover: #dc2626;
  --border: #334155;
  
  max-width: 600px;
  margin: 60px auto;
  padding: 30px;
  background-color: var(--bg-main);
  color: var(--text-main);
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  border-radius: 16px;
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.3), 0 10px 10px -5px rgba(0, 0, 0, 0.2);
}

/* Header Styles */
.dashboard-header {
  margin-bottom: 28px;
  text-align: center;
}
.dashboard-header h2 {
  font-size: 1.75rem;
  font-weight: 700;
  letter-spacing: -0.025em;
  margin: 0 0 6px 0;
}
.dashboard-subtitle {
  color: var(--text-muted);
  font-size: 0.9rem;
  margin: 0;
}

/* Card Styling */
.card {
  background-color: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 24px;
  margin-bottom: 32px;
}
.card h3 {
  margin-top: 0;
  margin-bottom: 18px;
  font-size: 1.1rem;
  font-weight: 600;
}

/* Layout Form Controls */
.form-group {
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.form-input {
  width: 100%;
  box-sizing: border-box;
  background-color: var(--bg-main);
  border: 1px solid var(--border);
  color: var(--text-main);
  padding: 12px 16px;
  border-radius: 8px;
  font-size: 0.95rem;
  transition: all 0.2s ease;
}
.form-input:focus {
  outline: none;
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.15);
}

/* Buttons UI */
.form-actions {
  margin-top: 6px;
}
.btn {
  width: 100%;
  padding: 12px;
  border: none;
  border-radius: 8px;
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.15s ease, transform 0.1s ease;
}
.btn:active {
  transform: scale(0.98);
}
.btn-primary { background-color: var(--primary); color: white; }
.btn-primary:hover { background-color: var(--primary-hover); }
.btn-success { background-color: var(--success); color: white; }
.btn-success:hover { background-color: var(--success-hover); }
.btn-secondary { background-color: var(--border); color: var(--text-main); }
.btn-secondary:hover { background-color: #475569; }

.btn-group {
  display: flex;
  gap: 10px;
}
.btn-group .btn {
  flex: 1;
}

/* Product Section & Counters */
.list-section h3 {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 1.1rem;
  margin-bottom: 16px;
}
.badge {
  background-color: var(--border);
  color: var(--text-muted);
  font-size: 0.8rem;
  padding: 2px 8px;
  border-radius: 20px;
}

/* Product List Loop UI */
.product-list {
  list-style-type: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.product-item {
  background-color: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 14px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.product-info {
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.product-name {
  font-weight: 600;
  font-size: 1rem;
}
.product-price {
  color: var(--success);
  font-size: 0.9rem;
  font-weight: 500;
}

/* Compact Actions Elements */
.product-actions {
  display: flex;
  gap: 8px;
}
.btn-icon {
  border: none;
  padding: 6px 12px;
  font-size: 0.85rem;
  font-weight: 500;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.15s ease;
}
.btn-edit { background-color: rgba(245, 158, 11, 0.15); color: #fbbf24; }
.btn-edit:hover { background-color: rgba(245, 158, 11, 0.25); }
.btn-delete { background-color: rgba(239, 68, 68, 0.15); color: #f87171; }
.btn-delete:hover { background-color: rgba(239, 68, 68, 0.25); }

.empty-state {
  text-align: center;
  padding: 30px;
  color: var(--text-muted);
  border: 2px dashed var(--border);
  border-radius: 8px;
}

/* Dynamic Vue Animations */
.list-enter-active, .list-leave-active {
  transition: all 0.3s ease;
}
.list-enter, .list-leave-to {
  opacity: 0;
  transform: translateY(10px);
}
</style>