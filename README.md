import { Routes, Route } from "react-router-dom";

import Navbar from "./components/Navbar";
import Home from "./pages/Home";
import Products from "./pages/Products";
import ProductDetail from "./pages/ProductDetail";

function App() {
  return (
    <>
      <Navbar />

      <main>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/products" element={<Products />} />
          <Route path="/products/:id" element={<ProductDetail />} />
        </Routes>
      </main>
    </>
  );
}

export default App;
function Home() {
  return (
    <section>
      <h1>Gestión de Productos</h1>

      <p>
        Aplicación desarrollada con React para consumir productos desde
        DummyJSON.
      </p>
    </section>
  );
}

export default Home;
import { Link } from "react-router-dom";

function Navbar() {
  return (
    <nav>
      <h2>React Products</h2>

      <ul>
        <li>
          <Link to="/">Inicio</Link>
        </li>

        <li>
          <Link to="/products">Productos</Link>
        </li>
      </ul>
    </nav>
  );
}

export default Navbar;
import { Link } from "react-router-dom";

function ProductCard({ product, onEdit, onDelete }) {
  return (
    <article className="product-card">
      <img
        src={product.thumbnail}
        alt={product.title}
      />

      <h3>{product.title}</h3>

      <p>{product.category}</p>

      <strong>${product.price}</strong>

      <div className="actions">
        <Link to={`/products/${product.id}`}>
          Ver detalle
        </Link>

        <button onClick={() => onEdit(product)}>
          Editar
        </button>

        <button onClick={() => onDelete(product.id)}>
          Eliminar
        </button>
      </div>
    </article>
  );
}

export default ProductCard;
import { useEffect, useState } from "react";
import { Link, useParams } from "react-router-dom";

import { getProductById } from "../services/productService";

function ProductDetail() {
  const { id } = useParams();

  const [product, setProduct] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");

  useEffect(() => {
    async function loadProduct() {
      try {
        const data = await getProductById(id);

        setProduct(data);
      } catch (error) {
        setError(error.message);
      } finally {
        setLoading(false);
      }
    }

    loadProduct();
  }, [id]);

  if (loading) {
    return <p>Cargando producto...</p>;
  }

  if (error) {
    return <p>{error}</p>;
  }

  return (
    <section className="product-detail">
      <Link to="/products">
        ← Regresar
      </Link>

      <h1>{product.title}</h1>

      <img
        src={product.thumbnail}
        alt={product.title}
      />

      <p>{product.description}</p>

      <p>
        <strong>Categoría:</strong> {product.category}
      </p>

      <p>
        <strong>Precio:</strong> ${product.price}
      </p>

      <p>
        <strong>Stock:</strong> {product.stock}
      </p>

      <p>
        <strong>Rating:</strong> {product.rating}
      </p>
    </section>
  );
}

export default ProductDetail;
import ProductCard from "./ProductCard";

function ProductList({ products, onEdit, onDelete }) {
  if (products.length === 0) {
    return <p>No se encontraron productos.</p>;
  }

  return (
    <section className="product-grid">
      {products.map((product) => (
        <ProductCard
          key={product.id}
          product={product}
          onEdit={onEdit}
          onDelete={onDelete}
        />
      ))}
    </section>
  );
}

export default ProductList;
import { useEffect, useState } from "react";

import ProductList from "../components/ProductList";

import {
  getProducts,
  searchProducts,
  getCategories,
  getProductsByCategory,
  createProduct,
  updateProduct,
  deleteProduct,
} from "../services/productService";

function Products() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");
  const [search, setSearch] = useState("");
  const [categories, setCategories] = useState([]);
  const [category, setCategory] = useState("");
  const [editingId, setEditingId] = useState(null);
  const [page, setPage] = useState(1);
  const [total, setTotal] = useState(0);

  const limit = 12;

  const [form, setForm] = useState({
    title: "",
    price: "",
    description: "",
  });

  useEffect(() => {
    loadProducts();
    loadCategories();
  }, []);

  async function loadCategories() {
    try {
      const data = await getCategories();

      setCategories(data);
    } catch (error) {
      console.error(error);
    }
  }

  async function loadProducts(pageNumber = 1) {
    try {
      setLoading(true);
      setError("");

      const skip = (pageNumber - 1) * limit;

      const data = await getProducts(limit, skip);

      setProducts(data.products);
      setTotal(data.total);

    } catch (error) {
      setError(error.message);

    } finally {
      setLoading(false);
    }
  }

  function nextPage() {
    const totalPages = Math.ceil(total / limit);

    if (page < totalPages) {
      const newPage = page + 1;

      setPage(newPage);
      loadProducts(newPage);
    }
  }

  function previousPage() {
    if (page > 1) {
      const newPage = page - 1;

      setPage(newPage);
      loadProducts(newPage);
    }
  }

  async function handleDelete(id) {
    try {
      await deleteProduct(id);

      setProducts(
        products.filter((product) => product.id !== id)
      );

    } catch (error) {
      setError(error.message);
    }
  }

  async function handleSearch(event) {
    event.preventDefault();

    try {
      setLoading(true);
      setCategory("");
      setPage(1);

      if (search.trim() === "") {
        await loadProducts();
        return;
      }

      const data = await searchProducts(search);

      setProducts(data.products);
      setTotal(data.total);

    } catch (error) {
      setError(error.message);

    } finally {
      setLoading(false);
    }
  }

  async function handleCategory(event) {
    const selectedCategory = event.target.value;

    setCategory(selectedCategory);
    setSearch("");
    setPage(1);

    try {
      setLoading(true);

      if (selectedCategory === "") {
        await loadProducts();
        return;
      }

      const data =
        await getProductsByCategory(selectedCategory);

      setProducts(data.products);
      setTotal(data.total);

    } catch (error) {
      setError(error.message);

    } finally {
      setLoading(false);
    }
  }

  function handleChange(event) {
    const { name, value } = event.target;

    setForm({
      ...form,
      [name]: value,
    });
  }

  async function handleCreate(event) {
    event.preventDefault();

    try {
      const newProduct = await createProduct({
        ...form,
        price: Number(form.price),
      });

      // DummyJSON siempre responde con el mismo id; se usa uno único
      // para que cada producto nuevo tenga su propia key en React.
      setProducts([
        { ...newProduct, id: Date.now() },
        ...products,
      ]);

      setForm({
        title: "",
        price: "",
        description: "",
      });

    } catch (error) {
      setError(error.message);
    }
  }

  function handleEdit(product) {
    setEditingId(product.id);

    setForm({
      title: product.title,
      price: product.price,
      description: product.description,
    });
  }

  async function handleUpdate(event) {
    event.preventDefault();

    try {
      const updated = await updateProduct(
        editingId,
        {
          ...form,
          price: Number(form.price),
        }
      );

      setProducts(
        products.map((product) =>
          product.id === editingId
            ? { ...product, ...updated }
            : product
        )
      );

      setEditingId(null);

      setForm({
        title: "",
        price: "",
        description: "",
      });

    } catch (error) {
      setError(error.message);
    }
  }

  if (loading) {
    return <p>Cargando productos...</p>;
  }

  if (error) {
    return <p>{error}</p>;
  }

  return (
    <section>
      <h1>Productos</h1>

      <form onSubmit={handleSearch} className="search-form">
        <input
          type="text"
          placeholder="Buscar producto..."
          value={search}
          onChange={(event) => setSearch(event.target.value)}
        />

        <button type="submit">
          Buscar
        </button>
      </form>

      <select
        value={category}
        onChange={handleCategory}
      >
        <option value="">
          Todas las categorías
        </option>

        {categories.map((item) => {
          const value = item.slug ?? item;
          const label = item.name ?? item;

          return (
            <option
              key={value}
              value={value}
            >
              {label}
            </option>
          );
        })}
      </select>

      <h2>{editingId ? "Editar producto" : "Agregar producto"}</h2>

      <form
        onSubmit={editingId ? handleUpdate : handleCreate}
        className="product-form"
      >
        <input
          type="text"
          name="title"
          placeholder="Nombre"
          value={form.title}
          onChange={handleChange}
          required
        />

        <input
          type="number"
          name="price"
          placeholder="Precio"
          value={form.price}
          onChange={handleChange}
          required
        />

        <textarea
          name="description"
          placeholder="Descripción"
          value={form.description}
          onChange={handleChange}
        />

        <button type="submit">
          {editingId
            ? "Actualizar"
            : "Guardar"}
        </button>
      </form>

      <ProductList
        products={products}
        onEdit={handleEdit}
        onDelete={handleDelete}
      />

      <div className="pagination">
        <button
          onClick={previousPage}
          disabled={page === 1}
        >
          Anterior
        </button>

        <span>
          Página {page}
        </span>

        <button
          onClick={nextPage}
          disabled={page >= Math.ceil(total / limit)}
        >
          Siguiente
        </button>
      </div>
    </section>
  );
}

export default Products;
import js from "@eslint/js";
import globals from "globals";
import reactHooks from "eslint-plugin-react-hooks";
import reactRefresh from "eslint-plugin-react-refresh";
import { defineConfig, globalIgnores } from "eslint/config";

export default defineConfig([
  globalIgnores(["dist"]),
  {
    files: ["**/*.{js,jsx}"],
    extends: [
      js.configs.recommended,
      reactHooks.configs.flat.recommended,
      reactRefresh.configs.vite,
    ],
    languageOptions: {
      ecmaVersion: "latest",
      globals: globals.browser,
      parserOptions: {
        ecmaFeatures: { jsx: true },
        sourceType: "module",
      },
    },
    rules: {
      "no-unused-vars": ["error", { varsIgnorePattern: "^[A-Z_]" }],
    },
  },
]);
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import "./styles/main.css";
import App from "./App.jsx";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>
);
{
  "name": "react-products",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "lint": "eslint .",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^19.3.1",
    "react-dom": "^19.3.1",
    "react-router-dom": "^7.18.5"
  },
  "devDependencies": {
    "@eslint/js": "^10.0.1",
    "@types/react": "^19.3.0",
    "@types/react-dom": "^19.3.0",
    "@vitejs/plugin-react": "^6.1.2",
    "eslint": "^10.14.0",
    "eslint-plugin-react-hooks": "^7.1.1",
    "eslint-plugin-react-refresh": "^0.5.7",
    "globals": "^17.13.0",
    "vite": "^8.3.5"
  }
}
const API_URL = "https://dummyjson.com/products";

// Obtener productos
export async function getProducts(limit = 12, skip = 0) {
  const response = await fetch(
    `${API_URL}?limit=${limit}&skip=${skip}`
  );

  if (!response.ok) {
    throw new Error("Error al obtener los productos");
  }

  return await response.json();
}

// Obtener producto por ID
export async function getProductById(id) {
  const response = await fetch(`${API_URL}/${id}`);

  if (!response.ok) {
    throw new Error("Producto no encontrado");
  }

  return await response.json();
}

// Buscar productos
export async function searchProducts(query) {
  const response = await fetch(
    `${API_URL}/search?q=${encodeURIComponent(query)}`
  );

  if (!response.ok) {
    throw new Error("Error al buscar productos");
  }

  return await response.json();
}

// Obtener categorías
export async function getCategories() {
  const response = await fetch(`${API_URL}/categories`);

  if (!response.ok) {
    throw new Error("Error al obtener las categorías");
  }

  return await response.json();
}

// Obtener productos por categoría
export async function getProductsByCategory(category) {
  const response = await fetch(
    `${API_URL}/category/${encodeURIComponent(category)}`
  );

  if (!response.ok) {
    throw new Error("Error al filtrar productos");
  }

  return await response.json();
}

// Crear producto
export async function createProduct(product) {
  const response = await fetch(`${API_URL}/add`, {
    method: "POST",

    headers: {
      "Content-Type": "application/json",
    },

    body: JSON.stringify(product),
  });

  if (!response.ok) {
    throw new Error("Error al crear el producto");
  }

  return await response.json();
}

// Actualizar producto
export async function updateProduct(id, product) {
  const response = await fetch(`${API_URL}/${id}`, {
    method: "PUT",

    headers: {
      "Content-Type": "application/json",
    },
 
    body: JSON.stringify(product),
  });

  if (!response.ok) {
    throw new Error("Error al actualizar el producto");
  }

  return await response.json();
}

// Eliminar producto
export async function deleteProduct(id) {
  const response = await fetch(`${API_URL}/${id}`, {
    method: "DELETE",
  });

  if (!response.ok) {
    throw new Error("Error al eliminar el producto");
  }

  return await response.json();
}
