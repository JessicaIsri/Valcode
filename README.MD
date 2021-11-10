# PROJETO INTEGRADOR 2020.2 - FATEC & SPC BRASIL - CADASTRO POSITIVO
 
# Resumo do projeto desenvolvido
Esse projeto foi desenvolvido visando a capacidade de gerar valor aos usuários do Cadastro Positivo, clientes do SPC BRASIL.
O Valcode é uma plataforma que tem como objetivo auxiliar na obtenção de dados sobre o seu cadastro positivo, onde era possivél consultar seu score, saber quantas vezes o seu crédito foi consultado, a evolução do seu score com o passar do tempo, juntar pontos de recompensas para o icentivo da não inadiplencia e acessar a dicas de como ter uma vida financeira mais saudavél.
Os dados utilizados foram capturados a partir de fontes publicas  e de caráter público, sobre o histórico de crédito, dessa forma podendo ser utilizados diante do consentimento do usuário.
Um destaque importante a se fazer é que há uma grande quantia de pessoas que raramente consta na base de dados, por não participar de aquisições sucetíveis ás consultas de crédito, os nomeados "desbancarizados", compostos geralmente de pessoas que não possuem contas bancarias. 
Sendo assim o sistema deve ser capaz de aceitar fontes de dados externas para além das fornecidas pelo SPC.
![image](https://user-images.githubusercontent.com/65822756/141018970-18faf248-3c21-4748-85a4-e09849948422.png)


# Técnologias utilizadas na solução

Java
Spring Boot
Angular
Postman

# Contribuições individuais/pessoais
## Desenvolimento dos Entity e Repositories: 
Os Entity e Repositories são classes de abstração das tabelas do banco de dados. A partir deles podemos acessar as colunas e seus respectivos dados. Para esse projeto foram utilizados as seguintes entidades:
- Fonte
- Modalidade
- Movimentos
- Operacoes
- Pagamentos
- PessoaFisica

```
import javax.persistence.*;
@AllArgsConstructor
@NoArgsConstructor
@Data
@Getter
@Setter
@Entity
@Table(name = "FONTE")
public class Fonte {

    @Id
    private Integer id;

    @Column(length = 50, nullable = false, name = "FONTE_NOME_COMERCIAL")
    private String fonte_nome;
}
```
Acima temos o exemplo de uma entidade referente a tabela fonte, explicando os elementos do código temos:
@AllArgsConstructor e @NoArgsConstructor são anotações pertencentes ao lombok que respectivamente criam um construto com todos os argumentos e um construtor sem os argumentos.

@Getter, @Setter, @Data são anotações pertencentes ao lombok que geram os getters e setters da classe.

@Entity sinaliza ao springboot que se trata de um componente de entidade.

@Table permite que indiquemos o nome da tabela que será referenciado pela classe em questão.

@Id indica que esse atributo se trata de uma coluna identificadora.

@Column permite que indiquemos os atributos da coluna, tais como o tamanho, se permite dado nulo o nome e etc (Muito util quando usamos o próprio hibernate para gerar os schemas.
```
package com.ExampleValcode.valcode.model.repository;

import com.ExampleValcode.valcode.model.entity.Fonte;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface FonteRepository extends JpaRepository<Fonte, Integer> {
}
```
Acima temos o repository da tabela, o mesmo extende os metódos da interface JpaRepository, para isso também precisamos passar qual entidade estamos utilizando e o tipo do seu campo identificador. 

## Serviços de Leitura dos csv's de entrada
Para além da maneira padrão de inserção através de crud, contavamos com um sistema de leitura de csv, que importava os dados para o banco. Segue abaixo um exemplo com a inserção dos dados de fonte:
```
@Service
public class CsvFonteService {


    private final FonteRepository repository;

    @Autowired
    public CsvFonteService(FonteRepository repository){
        this.repository = repository;
    }

    public void save(MultipartFile file){
        try{
            List<Fonte> fontes = CsvFonteHelper.csvToFonte(file.getInputStream());
            repository.saveAll(fontes);
        } catch (IOException e){
            throw new RuntimeException("Fail to Store csv data " + e.getMessage());
        }
    }

    public List<Fonte> getAllFontes(){
        return repository.findAll();
    }
}
```

O @Service indica ao springboot que se trata de um componente que contem as regras de negocio de uma determinada atividade, assim seno possivél o uso do @Autowired dessa classe. Iniciamos a mesma com o constructor que recebe o repository que sera utilizado para salvar os dados na base.
Em seguida temos o metódo save, ele ira utilizar a classe CsvFonteHelper para fazer a conversão do InputStream para um objeto da classe Fonte, a partir o uso de um buffer, percorrendo linha a linha e realizando o link entre a coluna do csv e a coluna do banco de dados:
```
public class CsvFonteHelper {
    public static String TYPE = "text/csv";
    public final String[] HEADERs = {"id", "nom_comercial"};
    public static List<Fonte> csvToFonte(InputStream is) {

        try (BufferedReader fileReader = new BufferedReader(new InputStreamReader(is, "UTF-8"));
             CSVParser csvParser = new CSVParser(fileReader,
                     CSVFormat.DEFAULT.withFirstRecordAsHeader().withIgnoreHeaderCase().withTrim());) {

            List<Fonte> fontes = new ArrayList<Fonte>();

            Iterable<CSVRecord> csvRecords = csvParser.getRecords();

            for (CSVRecord csvRecord : csvRecords) {
                Fonte fonte = new Fonte(
                        Integer.parseInt(csvRecord.get("id")),
                        csvRecord.get("nom_comercial")
                );

                fontes.add(fonte);
            }

            return fontes;
        } catch (IOException e) {
            throw new RuntimeException("fail to parse CSV file: " + e.getMessage());
        }
    }

}
```

Feito isso cada objeto é adicionado a um array que será retornado para o CsvFonteService e salvo a partir da instrução "saveAll()". A vantagem do "saveAll" em relação ao "save()" é que ao contrario do save que cria uma transação para cada registro, o saveAll utiliza uma unica transação para salva multiplos registros, dessa forma sendo muito mais perfomatico quando se é necessario salvar uma coleção de varias linhas no banco de dados.





# Aprendizados Efetivos
