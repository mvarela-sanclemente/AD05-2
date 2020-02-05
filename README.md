# Persistir un arquivo en PostgreSQL
## Creación da táboa
Imos comezas por crear a táboa que conterá o arquivo binario. O tipo de datos que usaremos é [**bytea**](https://www.postgresql.org/docs/9.0/datatype-binary.html). A táboa usarémola neste caso para gardar imaxes, pero podería ser calquera outro arquivo binario.

```java
//Creamos a táboa que conterá as imaxes
//NOTA: nón é moi lóxico crear funcións dende código. Só o fago para despois utilizala
String sqlTableCreation = new String(
    "CREATE TABLE IF NOT EXISTS imaxes (nomeimaxe text, img bytea);");

//Executamos a sentencia SQL anterior
CallableStatement createFunction = conn.prepareCall(sqlTableCreation);
createFunction.execute();
createFunction.close();
```
## Inserción dunha imaxe
Para facer un insert na táboa necesitaremos ler os bytes dun arquivo (vímolo como se realiza na unidade 2) e posteriormente insertala na base de datos.

```java
//Collemos o arquivo
String nomeFicheiro = new String("logo.png");
File file = new File(nomeFicheiro);
FileInputStream fis = new FileInputStream(file);

//Creamos a consulta que inserta a imaxe na base de datos
String sqlInsert = new String(
    "INSERT INTO imaxes VALUES (?, ?);");
PreparedStatement ps = conn.prepareStatement(sqlInsert);

//Engadimos como primeiro parámetro o nome do arquivo
ps.setString(1, file.getName());

//Engadimos como segundo parámetro o arquivo e a súa lonxitude
ps.setBinaryStream(2, fis, (int)file.length());

//Executamos a consulta
ps.executeUpdate();

//Cerrramos a consulta e o arquivo aberto
ps.close();
fis.close();
```
## Recuperación dun arquivo binario
Neste caso realizaremos unha consulta para recuperar o arquivo que intruducimos no paso anterior. Posteriormente gardaremos os bytes que se nos proporciona tal e como vimos na unidade 2. 

```java
//Creamos a consulta para recuperar a imaxe anterior
String sqlGet = new String(
    "SELECT img FROM imaxes WHERE nomeimaxe = ?;");
PreparedStatement ps2 = conn.prepareStatement(sqlGet); 

//Engadimos o nome da imaxe que queremos recuperar
ps2.setString(1, nomeFicheiro); 

//Executamos a consulta
ResultSet rs = ps2.executeQuery();

//Imos recuperando todos os bytes das imaxes
byte[] imgBytes = null;
while (rs.next()) 
{ 
    imgBytes = rs.getBytes(1); 
}

//Cerramos a consulta
rs.close(); 
ps2.close();

//Creamos o fluxo de datos para gardar o arquivo recuperado
String ficheiroSaida = new String("logo2.png");
File fileOut = new File(ficheiroSaida);
FileOutputStream fluxoDatos = new FileOutputStream(fileOut);

//Gardamos o arquivo recuperado
if(imgBytes != null){
    fluxoDatos.write(imgBytes);
}

//cerramos o fluxo de datos de saida
fluxoDatos.close();  
```

## Imaxes grandes
Se as imaxes que queremos gardar son moi grandes podemos utilizar outro tupi de datos en postgreSQL (imageoid). Tedes a documentación de como se realiza no seguinte enlace: [https://jdbc.postgresql.org/documentation/head/binary-data.html#binary-data-example](https://jdbc.postgresql.org/documentation/head/binary-data.html#binary-data-example)


