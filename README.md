<?php
// Conexión a la base de datos
$serverName = "data.ingesoft.com.py"; // Cambia según tu servidor
$connectionOptions = array(
    "Database" => "javiercolman", // Cambia por el nombre de tu base de datos
    "Uid" => "javiercolman", // Cambia por tu usuario de SQL Server
    "PWD" => "4r3H1hiTfFCbN5l" // Cambia por tu contraseña
);

// Conectar al servidor de SQL Server
$conn = sqlsrv_connect($serverName, $connectionOptions);

if (!$conn) {
    echo "Error en la conexión: ";
    die(print_r(sqlsrv_errors(), true));  // Muestra el error detallado
}

// Consulta para obtener las sucursales
$querySucursales = "SELECT Id_Sucursal as IdSucursal, Descripcion as NombreSucursal FROM Sucursales";
$stmtSucursales = sqlsrv_query($conn, $querySucursales);

if ($stmtSucursales === false) {
    echo "Error en la consulta de sucursales: ";
    die(print_r(sqlsrv_errors(), true));  // Muestra el error detallado
}

$sucursales = [];
while ($row = sqlsrv_fetch_array($stmtSucursales, SQLSRV_FETCH_ASSOC)) {
    $sucursales[] = $row;
}

// Consulta para obtener la lista de productos
$queryProductos = "SELECT UniqueId, Pnombre, PrecioU FROM Productos";
$stmtProductos = sqlsrv_query($conn, $queryProductos);

if ($stmtProductos === false) {
    echo "Error en la consulta SQL: ";
    die(print_r(sqlsrv_errors(), true));  // Muestra el error detallado
}

$productos = [];
while ($row = sqlsrv_fetch_array($stmtProductos, SQLSRV_FETCH_ASSOC)) {
    $productos[] = $row;
}
?>

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Formulario de Carga de Productos</title>
    <script>
        // Función para calcular las columnas y el totalizador
        function calcularVenta(index) {
            var entrada = parseFloat(document.getElementById('entrada' + index).value) || 0;
            var recarga = parseFloat(document.getElementById('recarga' + index).value) || 0;
            var averiado = parseFloat(document.getElementById('averiado' + index).value) || 0;
            var cierre = parseFloat(document.getElementById('cierre' + index).value) || 0;

            // Cálculo de la columna "Venta"
            var venta = entrada + recarga - averiado - cierre;
            document.getElementById('venta' + index).value = venta;

            // Cálculo de la columna "Venta * Precio"
            var precioVenta = parseFloat(document.getElementById('precio_venta' + index).value) || 0;
            var total = venta * precioVenta;
            document.getElementById('total' + index).value = total;

            // Calcular el total acumulado
            actualizarTotalizador();
        }

        // Función para actualizar el totalizador de la columna Total
		function actualizarTotalizador() {
			var totalizador = 0;
			var totalInputs = document.querySelectorAll('input[id^="total"]');
			
			totalInputs.forEach(function(input) {
				totalizador += parseFloat(input.value) || 0;
			});
			
			// Formatear el totalizador con separador de miles
			var totalConSeparador = totalizador.toLocaleString('es-ES', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
			
			document.getElementById('totalizador').innerText = 'Total: ' + totalConSeparador;
		}


        // Función para limpiar los campos editables
        function limpiarCampos() {
            let confirmar = confirm("¿Estás seguro de refrescar todos los campos?");
            if (confirmar) {
                location.reload();  // Refrescar la página
            }
        }
    </script>
    <style>
        body {
            background-color: #f8f9fa;
            font-family: 'Arial', sans-serif;
        }

        .container {
            margin-top: 50px;
            padding: 30px;
            background-color: #fff;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        }

        h2 {
            text-align: center;
            color: #343a40;
            margin-bottom: 30px;
        }

        table {
            width: 100%;
            border-collapse: collapse;
        }

        th, td {
            padding: 10px;
            text-align: center;
            border: 1px solid #ddd;
        }

        th {
            background-color: #f39c12; /* Mostaza */
            color: white;
        }

        td input {
            width: 100%;
            padding: 5px;
            font-size: 14px;
            border: 1px solid #ddd;
            border-radius: 5px;
        }

        td input[readonly] {
            background-color: #f0f0f0;
        }

        td input:focus {
            outline: none;
            border-color: #007bff;
        }

        button {
            background-color: #f39c12; /* Mostaza */
            color: white;
            padding: 10px 20px;
            font-size: 16px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            width: 100%;
            margin-top: 20px;
        }

        button:hover {
            background-color: #e67e22; /* Naranja */
        }

        .limpiar-btn {
            background-color: #e74c3c; /* Rojo */
            color: white;
            padding: 10px 20px;
            font-size: 16px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin-bottom: 20px;
        }

        .limpiar-btn:hover {
            background-color: #c0392b; /* Rojo oscuro */
        }

        .totalizador {
            font-size: 24px;
            font-weight: bold;
            text-align: right;
            margin-top: 20px;
        }
    </style>
</head>
<body>

<div class="container">
    <h2>BIENVENIDO</h2>
	<h2>FORMULARIO DE PRODUCTOS</h2>

    <!-- Combos para seleccionar sucursal y turno -->
    <form action="guardar_datos.php" method="POST">
        <label for="sucursal">SUCURSALES:</label>
        <select id="sucursal" name="sucursal" required>
            <option value="">SELECCIONA UNA SUCURSAL:</option>
            <?php foreach ($sucursales as $sucursal): ?>
                <option value="<?= $sucursal['IdSucursal'] ?>"><?= $sucursal['NombreSucursal'] ?></option>
            <?php endforeach; ?>
        </select>

        <label for="turno">TURNO:</label>
        <select id="turno" name="turno" required>
            <option value="mañana">MAÑANA</option>
            <option value="tarde">TARDE</option>
        </select>

        <br><br>

        <!-- Botón para limpiar campos editables -->
        <button type="button" class="limpiar-btn" onclick="limpiarCampos()">Limpiar Campos</button>

        <table>
            <thead>
                <tr>
                    <th>UniqueId</th>
                    <th>Producto</th>
                    <th>Precio Venta</th>
                    <th>Entrada</th>
                    <th>Recarga</th>
                    <th>Averiado</th>
                    <th>Cierre</th>
                    <th>Venta</th>
                    <th>Total</th>
                </tr>
            </thead>
            <tbody>
                <?php
                // Mostrar productos en el formulario
                if (isset($productos) && is_array($productos)) {
                    foreach ($productos as $index => $producto) {
                        echo "<tr>";
                        echo "<td>{$producto['UniqueId']}</td>"; // UniqueId
                        echo "<td>{$producto['Pnombre']}</td>"; // Nombre del Producto
                        echo "<td><input type='number' id='precio_venta{$index}' value='{$producto['PrecioU']}' readonly></td>"; // Precio Venta
                        echo "<td><input type='number' id='entrada{$index}' name='entrada[{$producto['UniqueId']}]' oninput='calcularVenta({$index})' required></td>";
                        echo "<td><input type='number' id='recarga{$index}' name='recarga[{$producto['UniqueId']}]' oninput='calcularVenta({$index})' required></td>";
                        echo "<td><input type='number' id='averiado{$index}' name='averiado[{$producto['UniqueId']}]' oninput='calcularVenta({$index})' required></td>";
                        echo "<td><input type='number' id='cierre{$index}' name='cierre[{$producto['UniqueId']}]' oninput='calcularVenta({$index})' required></td>";
                        echo "<td><input type='number' id='venta{$index}' name='venta[{$producto['UniqueId']}]' readonly></td>"; // Venta
                        echo "<td><input type='text' id='total{$index}' name='total[{$producto['UniqueId']}]' value='" . number_format(0, 0, ',', '.') . "' readonly></td>"; // Total con separador de miles

                        echo "</tr>";
                    }
                } else {
                    echo "<tr><td colspan='9'>No se encontraron productos</td></tr>";
                }
                ?>
            </tbody>
        </table>

        <!-- Totalizador -->
        <div class="totalizador" id="totalizador">Total: 0.00</div>

        <button type="submit">Guardar Datos</button>
    </form>
</div>

</body>
</html>
