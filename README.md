<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8" />
<title>Sistema ERP - Peças de Celular</title>

<!-- React e Babel via CDN -->
<script src="https://unpkg.com/react@18/umd/react.development.js"></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

<style>
:root {
  --bg: #f3f4f6;
  --white: #ffffff;
  --muted: #6b7280;
  --accent: #2563eb;
}

* { box-sizing: border-box; font-family: Arial, sans-serif; }

body, html { margin: 0; background: var(--bg); height: 100%; }

.app { display: flex; height: 100vh; }

.sidebar {
  width: 240px;
  background: var(--white);
  padding: 20px;
  border-right: 1px solid #e5e7eb;
}

.sidebar h2 { margin-bottom: 12px; }

.sidebar ul { padding: 0; list-style: none; }
.sidebar li { padding: 10px 0; cursor: pointer; color: var(--muted); }
.sidebar li.active { color: var(--accent); font-weight: 700; }

.main { flex: 1; padding: 24px; overflow-y: auto; }

.searchWrap {
  display: flex; gap: 10px; align-items: center;
}

.searchWrap input {
  padding: 10px;
  border-radius: 8px;
  border: 1px solid #d1d5db;
  width: 300px;
}

.tableSection {
  background: var(--white);
  padding: 16px;
  border-radius: 10px;
  margin-top: 20px;
}

table { width: 100%; border-collapse: collapse; }
th, td { padding: 10px; border-bottom: 1px solid #f3f4f6; }
th { background: #fafafa; color: var(--muted); }

.product-form { display: flex; flex-direction: column; gap: 12px; max-width: 500px; }
.product-form input {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 8px;
}
.product-form button {
  padding: 10px;
  background: var(--accent);
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
}
</style>

</head>
<body>

<div id="root"></div>

<script type="text/babel">

// -------------------- FUNÇÕES DE NORMALIZAÇÃO --------------------
function normalize(text) {
  return text
    .toLowerCase()
    .normalize("NFD").replace(/\p{Diacritic}/gu, "")
    .replace(/[^a-z0-9 ]/g, " ")
    .replace(/\s+/g, " ")
    .trim();
}

function makeSearchIndex(p) {
  return normalize(`${p.name} ${p.code} ${p.brand} ${p.category}`);
}

// -------------------- COMPONENTE FORMULÁRIO --------------------
function ProductForm({ onSave }) {

  const [product, setProduct] = React.useState({
    code: "",
    name: "",
    brand: "",
    category: "",
    stock: 0,
    cost: 0
  });

  function change(e) {
    const { name, value } = e.target;
    setProduct(p => ({
      ...p,
      [name]: (name === "stock" || name === "cost") ? Number(value) : value
    }));
  }

  function save(e) {
    e.preventDefault();
    if (!product.name) return alert("Nome é obrigatório!");
    onSave(product);
    setProduct({ code: "", name: "", brand: "", category: "", stock: 0, cost: 0 });
  }

  return (
    <form onSubmit={save} className="product-form">

      <input placeholder="Código" name="code" value={product.code} onChange={change} />

      <input placeholder="Nome do produto *" name="name" value={product.name} onChange={change} required />

      <input placeholder="Marca" name="brand" value={product.brand} onChange={change} />

      <input placeholder="Categoria" name="category" value={product.category} onChange={change} />

      <input placeholder="Estoque" type="number" name="stock" value={product.stock} onChange={change} />

      <input placeholder="Preço (R$)" type="number" step="0.01" name="cost" value={product.cost} onChange={change} />

      <button type="submit">Salvar Produto</button>
    </form>
  );
}

// -------------------- SISTEMA PRINCIPAL --------------------
function App() {

  const [secao, setSecao] = React.useState("produtos");
  const [query, setQuery] = React.useState("");

  const [products, setProducts] = React.useState([
    { id: 1, code: "163-464", name: "Tela Samsung A50 Incell Original", category: "Telas", brand: "Samsung", stock: 12, cost: 120 },
    { id: 2, code: "163-1208", name: "Display A50 Tela Premium", category: "Telas", brand: "Samsung", stock: 8, cost: 150 },
    { id: 3, code: "163-1210", name: "A50 Tela Oled Completa", category: "Telas", brand: "Samsung", stock: 5, cost: 180 },
  ]);

  const indexed = React.useMemo(
    () => products.map(p => ({ ...p, searchIndex: makeSearchIndex(p) })),
    [products]
  );

  const results = React.useMemo(() => {
    const terms = normalize(query).split(" ").filter(Boolean);
    if (terms.length === 0) return indexed;

    return indexed.filter(p =>
      terms.every(t => p.searchIndex.includes(t))
    );
  }, [query, indexed]);

  function saveProduct(p) {
    const id = Math.max(0, ...products.map(p => p.id)) + 1;
    setProducts(prev => [...prev, { id, ...p }]);
    setSecao("produtos");
  }

  return (
    <div className="app">

      <aside className="sidebar">
        <h2>Menu</h2>
        <ul>
          <li className={secao === "produtos" ? "active" : ""} onClick={() => setSecao("produtos")}>Produtos</li>
          <li className={secao === "cadastro" ? "active" : ""} onClick={() => setSecao("cadastro")}>Cadastro</li>
        </ul>
      </aside>

      <main className="main">

        {secao === "produtos" && (
          <>
            <h1>Produtos</h1>

            <div className="searchWrap">
              <input
                placeholder="Buscar: ex 'tela a50'"
                value={query}
                onChange={e => setQuery(e.target.value)}
              />
            </div>

            <div className="tableSection">
              <table>
                <thead>
                  <tr>
                    <th>Código</th>
                    <th>Nome</th>
                    <th>Categoria</th>
                    <th>Marca</th>
                    <th>Estoque</th>
                    <th>Preço</th>
                  </tr>
                </thead>

                <tbody>
                  {results.map(p => (
                    <tr key={p.id}>
                      <td>{p.code}</td>
                      <td>{p.name}</td>
                      <td>{p.category}</td>
                      <td>{p.brand}</td>
                      <td>{p.stock}</td>
                      <td>R$ {p.cost.toFixed(2)}</td>
                    </tr>
                  ))}

                  {results.length === 0 && (
                    <tr>
                      <td colSpan="6" style={{ textAlign: "center" }}>
                        Nenhum produto encontrado.
                      </td>
                    </tr>
                  )}
                </tbody>

              </table>
            </div>
          </>
        )}

        {secao === "cadastro" && (
          <>
            <h1>Cadastro de Produtos</h1>
            <ProductForm onSave={saveProduct} />
          </>
        )}

      </main>

    </div>
  );
}

// Renderização
ReactDOM.createRoot(document.getElementById("root")).render(<App/>);

</script>

</body>
</html>
