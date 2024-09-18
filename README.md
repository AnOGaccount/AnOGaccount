<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Butcher Shop Inventory</title>

  <!-- React and ReactDOM -->
  <script src="https://unpkg.com/react@17/umd/react.development.js"></script>
  <script src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>

  <!-- Babel for JSX support -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/6.26.0/babel.min.js"></script>

  <!-- jsPDF and jsPDF AutoTable for generating PDFs -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.17/jspdf.plugin.autotable.min.js"></script>

  <style>
    body { font-family: Arial, sans-serif; padding: 20px; }
    table { width: 100%; border-collapse: collapse; margin-bottom: 20px; }
    th, td { border: 1px solid black; padding: 8px; }
  </style>
</head>
<body>
  <div id="root"></div>

  <script type="text/babel">
    const { useState, useEffect } = React;

    const initialProducts = [
      { id: 1, name: 'Ribeye Steak', quantity: 50, reorderLevel: 20 },
      { id: 2, name: 'Ground Beef', quantity: 100, reorderLevel: 30 },
      { id: 3, name: 'Pork Chops', quantity: 75, reorderLevel: 25 },
      { id: 4, name: 'Chicken Breast', quantity: 120, reorderLevel: 40 },
      { id: 5, name: 'Lamb Chops', quantity: 30, reorderLevel: 15 },
    ];

    const ButcherInventoryApp = () => {
      const [products, setProducts] = useState(initialProducts);
      const [reorderReport, setReorderReport] = useState([]);

      useEffect(() => {
        generateReorderReport();
      }, [products]);

      const updateQuantity = (id, newQuantity) => {
        setProducts(products.map(product =>
          product.id === id ? { ...product, quantity: parseInt(newQuantity) || 0 } : product
        ));
      };

      const generateReorderReport = () => {
        const report = products
          .filter(product => product.quantity < product.reorderLevel)
          .map(product => ({
            ...product,
            reorderQuantity: product.reorderLevel - product.quantity,
          }));
        setReorderReport(report);
      };

      const downloadPdf = () => {
        const { jsPDF } = window.jspdf;
        const doc = new jsPDF();

        doc.text('Reorder Report', 14, 16); // Title

        if (reorderReport.length > 0) {
          const tableColumn = ['Product', 'Current Quantity', 'Reorder Level', 'Reorder Quantity'];
          const tableRows = [];

          reorderReport.forEach(product => {
            const productData = [
              product.name,
              product.quantity,
              product.reorderLevel,
              product.reorderQuantity,
            ];
            tableRows.push(productData);
          });

          doc.autoTable({
            head: [tableColumn],
            body: tableRows,
            startY: 22,
          });
        } else {
          doc.text('No products need reordering at this time.', 14, 22);
        }

        doc.save('reorder_report.pdf'); // Save the PDF
      };

      return (
        <div>
          <h1>Butcher Shop Inventory Management</h1>

          <h2>Current Inventory</h2>
          <table>
            <thead>
              <tr>
                <th>Product</th>
                <th>Quantity</th>
                <th>Reorder Level</th>
              </tr>
            </thead>
            <tbody>
              {products.map(product => (
                <tr key={product.id}>
                  <td>{product.name}</td>
                  <td>
                    <input
                      type="number"
                      value={product.quantity}
                      onChange={(e) => updateQuantity(product.id, e.target.value)}
                      min="0"
                    />
                  </td>
                  <td>{product.reorderLevel}</td>
                </tr>
              ))}
            </tbody>
          </table>

          <h2>Reorder Report</h2>
          {reorderReport.length > 0 ? (
            <table>
              <thead>
                <tr>
                  <th>Product</th>
                  <th>Current Quantity</th>
                  <th>Reorder Level</th>
                  <th>Reorder Quantity</th>
                </tr>
              </thead>
              <tbody>
                {reorderReport.map(product => (
                  <tr key={product.id}>
                    <td>{product.name}</td>
                    <td>{product.quantity}</td>
                    <td>{product.reorderLevel}</td>
                    <td>{product.reorderQuantity}</td>
                  </tr>
                ))}
              </tbody>
            </table>
          ) : (
            <p>No products need reordering at this time.</p>
          )}

          <button onClick={downloadPdf}>Download Reorder Report as PDF</button>
        </div>
      );
    };

    ReactDOM.render(<ButcherInventoryApp />, document.getElementById('root'));
  </script>
</body>
</html>
