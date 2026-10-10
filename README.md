<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Escáner de Código de Barras</title>
  <!-- CDN de Html5Qrcode para escaneo continuo con cámara -->
  <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
  <style>
    * { box-sizing: border-box; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
    body { margin: 0; padding: 16px; background-color: #f4f6f8; color: #333; }
    h1 { font-size: 1.4rem; text-align: center; margin-bottom: 16px; color: #1a1a1a; }
    .card { background: #fff; padding: 16px; border-radius: 12px; box-shadow: 0 2px 8px rgba(0,0,0,0.08); margin-bottom: 16px; }
    #reader { width: 100%; border-radius: 8px; overflow: hidden; background: #000; }
    .btn-group { display: flex; gap: 8px; margin-top: 12px; }
    button { flex: 1; padding: 12px; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; font-size: 0.95rem; }
    .btn-primary { background: #007aff; color: white; }
    .btn-danger { background: #ff3b30; color: white; }
    .btn-success { background: #34c759; color: white; }
    table { width: 100%; border-collapse: collapse; margin-top: 8px; }
    th, td { text-align: left; padding: 10px 8px; border-bottom: 1px solid #eee; font-size: 0.85rem; }
    th { background: #fafafa; font-weight: 600; }
    .badge { background: #e5f1ff; color: #007aff; padding: 4px 8px; border-radius: 4px; font-weight: bold; }
  </style>
</head>
<body>

  <h1>Registro de Código de Barras</h1>

  <!-- Tarjeta del Escáner -->
  <div class="card">
    <div id="reader"></div>
    <div class="btn-group">
      <button class="btn-primary" onclick="startScanner()">Iniciar Cámara</button>
      <button class="btn-danger" onclick="stopScanner()">Detener Cámara</button>
    </div>
  </div>

  <!-- Tarjeta de Controles y Tabla -->
  <div class="card">
    <div style="display: flex; justify-content: space-between; align-items: center;">
      <h3>Registros (<span id="total-count">0</span>)</h3>
      <div class="btn-group" style="margin-top: 0;">
        <button class="btn-success" onclick="exportToCSV()">Exportar CSV</button>
        <button class="btn-danger" onclick="clearData()">Borrar Todo</button>
      </div>
    </div>

    <table>
      <thead>
        <tr>
          <th>#</th>
          <th>Código</th>
          <th>Hora</th>
        </tr>
      </thead>
      <tbody id="log-table">
        <!-- Los datos se insertarán aquí -->
      </tbody>
    </table>
  </div>

  <script>
    let html5QrcodeScanner = null;
    let records = JSON.parse(localStorage.getItem('barcode_records') || '[]');

    function renderTable() {
      const tbody = document.getElementById('log-table');
      const countEl = document.getElementById('total-count');
      tbody.innerHTML = '';
      countEl.textContent = records.length;

      records.slice().reverse().forEach((item, index) => {
        const row = document.createElement('tr');
        row.innerHTML = `
          <td>${records.length - index}</td>
          <td><span class="badge">${item.code}</span></td>
          <td>${item.timestamp}</td>
        `;
        tbody.appendChild(row);
      });
    }

    function onScanSuccess(decodedText, decodedResult) {
      // Reproducir sonido/vibración si está disponible
      if (navigator.vibrate) navigator.vibrate(100);

      const now = new Date();
      const timeStr = now.toLocaleTimeString() + ' (' + now.toLocaleDateString() + ')';

      // Agregar registro al inicio
      records.push({ code: decodedText, timestamp: timeStr });
      localStorage.setItem('barcode_records', JSON.stringify(records));
      renderTable();
    }

    function startScanner() {
      if (!html5QrcodeScanner) {
        html5QrcodeScanner = new Html5Qrcode("reader");
      }
      
      const config = { fps: 10, qrbox: { width: 250, height: 150 } };
      
      html5QrcodeScanner.start(
        { facingMode: "environment" }, // Usa la cámara trasera
        config,
        onScanSuccess
      ).catch(err => {
        alert("Error al acceder a la cámara: " + err);
      });
    }

    function stopScanner() {
      if (html5QrcodeScanner) {
        html5QrcodeScanner.stop().then(() => {
          console.log("Escáner detenido");
        }).catch(err => console.error(err));
      }
    }

    function exportToCSV() {
      if (records.length === 0) {
        alert("No hay registros para exportar.");
        return;
      }
      let csvContent = "data:text/csv;charset=utf-8,Nro,Codigo,FechaHora\n";
      records.forEach((r, i) => {
        csvContent += `${i + 1},"${r.code}","${r.timestamp}"\n`;
      });
      const encodedUri = encodeURI(csvContent);
      const link = document.createElement("a");
      link.setAttribute("href", encodedUri);
      link.setAttribute("download", `registros_codigos_${new Date().toISOString().slice(0,10)}.csv`);
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    }

    function clearData() {
      if (confirm("¿Estás seguro de eliminar todos los registros?")) {
        records = [];
        localStorage.removeItem('barcode_records');
        renderTable();
      }
    }

    // Cargar tabla al iniciar
    renderTable();
  </script>
</body>
</html>
[index.html](https://github.com/user-attachments/files/33292575/index.html)
