<template>
  <div class="container-fluid w-50 p-4">
    <div class="card">
      <h3 class="beautify">Formulário de Cadastro de dados</h3>
      <input
        class="form-control"
        type="text"
        placeholder="Informe o nome"
        v-model="name"
        maxlength="20"
        required
      />
      <input
        class="form-control"
        type="text"
        placeholder="Informe o sobrenome"
        maxlength="30"
        v-model="sobre"
        required
      />
      <input
        class="form-control"
        type="text"
        placeholder="Informe a empresa"
        maxlength="20"
        v-model="empresa"
        required
      />
      <input
        class="form-control"
        type="number"
        placeholder="Telefone"
        maxlength="20"
        v-model="phone"
        required
      />
      <input
        class="form-control"
        type="text"
        placeholder="Email"
        maxlength="50"
        v-model="mail"
        required
      />

      <select class="form-control" required v-model="idType">
        <!-- <option selected>Selecione o tipo de contato</option> -->
        <option value="1">Público</option>
        <option value="2">Particular</option>
        <option value="3">Interno</option>
      </select>
      <button
        v-if="clickedEdit == false"
        class="btn btn-success btn-md"
        type="submit"
        @click="
          createRegistro(name, idType, userKey, sobre, empresa, phone, mail)
        "
        :disabled="loading"
      >
        <div v-if="loading">
          <div class="spinner-border text-light" role="status">
            <span class="sr-only"></span>
          </div>
        </div>
        <div v-else>{{ loading }}</div>
      </button>
      <button
        v-if="clickedEdit == true"
        class="btn btn-warning btn-md"
        type="submit"
        @click="
          createRegistro(name, idType, userKey, sobre, empresa, phone, mail)
        "
      >
        Atualizar Registro
      </button>

      <button class="btn btn-danger btn-md" type="submit" @click="clearReg">
        Resetar formulário
      </button>
      <button
        class="btn btn-success btn-sm"
        v-if="clickedEdit == true"
        type="submit"
        @click="reload()"
      >
        Voltar
      </button>
    </div>
    <!-- <p>Filtrar dados</p> -->

    <table class="table table-bordered table-hover table-dark">
      <thead>
        <tr>
          <th>User ID</th>
          <th>Nome</th>
          <th>Sobrenome</th>
          <th>Empresa</th>
          <th>Telefone</th>
          <th>Email</th>
          <th>Excluir dado</th>
          <th>Editar</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="dado in dados" :key="dado">
          <!--O primeiro item do VFOR funciona como um alias para realizar 
      a iteração de cada objeto presente no array no caso desse código. 
      O segundo item "x" é o nosso indíce (posição do arr)
      Para cada dado presente no array de dados, faça "x" indicado no <li>
      !!!!! A key "dado" é importante para identificar cada elemento unicamente no array
    -->
          <td>{{ dado.id }}</td>
          <td>{{ dado.nome }}</td>
          <td>{{ dado.sobrenome }}</td>
          <td>{{ dado.empresa }}</td>
          <td>{{ dado.telefone_movel }}</td>
          <td>{{ dado.email1 }}</td>
          <td>
            <button
              input
              type="submit"
              class="btn btn-primary btn-danger btn-sm"
              @submit.prevent=""
              @click="deletaRegistro(dado.id)"
            >
              Remover
            </button>
          </td>
          <td>
            <button
              input
              type="submit"
              class="btn btn-primary btn-info btn-sm editReg"
              @submit.prevent=""
              @click="editaReg(dado.id)"
            >
              Editar
            </button>
          </td>
        </tr>
      </tbody>
    </table>
    <div class="card border border-dark">
      <h3 class="card-title beautify">Buscar dados</h3>

      <input
        type="text"
        class="searchBar"
        placeholder="Digite sua busca"
        @keyup="debounce"
        v-model="search"
      />
    </div>
    <br /><br />
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      dados: [], //Array de objetos para receber e exibir os dados da API
      search: "", //Search é a var que sustenta a busca do usuário, atrelado ao V-MODEL
      timer: null, //Timer é a var que faz parte da função debounce para controlar o delay de input de cada user
      checkId: false, //CheckID é acionado no carregamento de dados pelo created()
      name: "",
      sobre: "",
      empresa: "",
      phone: null,
      mail: "",
      idType: "1",
      userKey: null,
      isPressed: false,
      clickedEdit: false,
      clearedWhileEditing: false,
      loading: false,
      //Variáveis de uso para passagem de parâmetros e v-model
    };
  },
  methods: {
    reload() {
      window.location.reload(true);
    },
    validaRequest() {
      //Função chamada no método executaRequest e verifica
      //se o input do user foi maior do que 3 caracteres para realizar um get na api
      if (this.search.length <= 3) {
        return false;
      }
      if (this.search.length > 3) {
        return true;
      }
    },
    debounce() {
      //Função debounce, chamada no template "onkeyup"
      if (this.timer) {
        //Se existe algo no timer, é realizado um clear para deletar a saída.
        clearTimeout(this.timer);
      }
      this.timer = setTimeout(() => {
        //Irá executar o código, no caso o search após os ms definidos abaixo
        this.executaRequest();
      }, 500);
    },
    executaRequest() {
      if (
        this.validaRequest() ||
        this.search.length == 0 ||
        this.checkId == true
      ) {
        //Se o search for > 3 ou se for == 0, realiza a request
        const self = this;
        const token =
          "a2279db699464d5026a46688f85e64cca895fe497d8926003f168ef86cd6ed74";
        //Passando o token para uma var
        axios({
          method: "get", //Definindo o método (poderia ser POST, PUT.. ETC)
          url:
            "https://voice.convert.app.br/assist/api/Contato/buscarAgendaUsuario/?formato=json&q=" +
            this.search,
          //Passando URL da api
          headers: {
            Authorization: "Bearer " + token,
          },
          //O header no axios consiste na passagem de objetos contendo nossos parâmetros definidos
          //de acordo com o que é requisitado para 'x' aplicação
        }).then(function (response) {
          //Usa-se o método then() para processar o resultado
          self.dados = response.data.data; //GET os dados presentes na API
        });
      }
    },
    deletaRegistro(userId) {
      const self = this;
      const token =
        "a2279db699464d5026a46688f85e64cca895fe497d8926003f168ef86cd6ed74";

      swal
        .fire({
          title: "Deseja remover o usuário?",
          icon: "warning",
          showCancelButton: true,
          showConfirmButton: true,
          confirmButtonColor: "green",
          cancelButtonColor: "red",
          confirmButtonText: "Sim",
          cancelButtonText: "Não",
        })
        .then((result) => {
          if (result.isConfirmed) {
            axios({
              method: "post",
              url: "https://voice.convert.app.br/assist/api/Contato/excluirContato/",
              headers: {
                Authorization: "Bearer " + token,
                "Content-Type": "application/x-www-form-urlencoded",
              },
              data: "id=" + userId, //Passamos o ID como parâmetro e excluímos o registro
            }).then(function (response) {
              console.log(response.data.status);
              if (response.data.status < 400) {
                self.executaRequest();
                self.checkId = true;
                swal.fire({
                  title: "Registro removido com sucesso!",
                  icon: "success",
                  showConfirmButton: true,
                });
              }
            });
          }
        });
    },
    clearReg() {
      if (this.clearedWhileEditing == true) {
        //Entra na condição caso o usuário tenha escolhido limpar o form durante a EDIÇÃO dos dados
        //No caso irá chamar o método editaReg para não perder o ID do usuário
        this.editaReg(this.userId);
        this.name = "";
        this.sobre = "";
        this.empresa = "";
        this.phone = "";
        this.mail = "";
        this.clearedWhileEditing = false;
      } else {
        this.name = "";
        this.sobre = "";
        this.empresa = "";
        this.phone = "";
        this.mail = "";
      }
      //Limpa os dados após inserção no form
    },
    createRegistro(name, idType, userId, sobre, empresa, phone, mail) {
      this.loading = true;
      //Recebe como parâmetros os dados para realizar o POST
      const self = this;
      const token =
        "a2279db699464d5026a46688f85e64cca895fe497d8926003f168ef86cd6ed74";

      if (this.isPressed == true) {
        //Tratamento da EDIÇÃO de Registro
        //O user pressionou o botão de editar? Quando verdade irá entrar nessa condição para realizar o update no ID escolhido

        this.clearedWhileEditing = true;
        if (name == "") {
          swal.fire("Você precisa informar um nome", "", "error");
          this.editaReg(userId); //Continua na função de editar (Evitando que caia na func de cadastro.)
          this.loading = false;
        } else {
          axios({
            method: "post",
            url: "https://voice.convert.app.br/assist/api/Contato/persistFromAssist/",
            headers: {
              Authorization: "Bearer " + token,
              "Content-Type": "application/x-www-form-urlencoded",
            },
            data: {
              //Data é o objeto que contém os dados que queremos passar na API.
              email1: mail,
              telefone_movel: phone,
              sobrenome: sobre,
              empresa: empresa,
              id: userId,
              id_tipo_contato: idType,
              nome: name,
            },
          })
            .then(function (response) {
              console.log(response.data);
              if (response.data.status >= 400) {
                swal.fire(
                  "Erro na edição dos seus dados ",
                  response.data.msg,
                  "error"
                );
              } else {
                self.executaRequest();
                self.checkId = true;
                swal.fire(
                  "Os dados foram atualizados.",
                  response.data.msg,
                  "success"
                );
                self.clearReg();
                self.clickedEdit = false; //Reseta a var clickedEdit
                self.isPressed = false; //Reseta a var isPressed para false, cada vez que o user pressionar no "Editar" receberá true
              }
            })
            .finally(() => (this.loading = false));
        }
      } else {
        //Condição de CADASTRO no form
        //Se o isPressed for false, o usuário vai cadastrar um usuário NOVO.
        if (name == "") {
          swal.fire("Seu cadastro precisa conter um nome.", "", "info");
          this.loading = false;
        } else {
          axios({
            method: "post",
            url: "https://voice.convert.app.br/assist/api/Contato/persistFromAssist",
            headers: {
              Authorization: "Bearer " + token,
              "Content-Type": "application/x-www-form-urlencoded",
            },
            data: {
              email1: mail,
              telefone_movel: phone,
              sobrenome: sobre,
              empresa: empresa,
              id_tipo_contato: idType,
              nome: name,
            },
          })
            .then(function (response) {
              console.log(response.data);
              console.log(response.data.status);
              if (response.data.status >= 400) {
                swal.fire("Erro no cadastro", response.data.msg, "error");
              } else {
                swal.fire(
                  "Os dados foram cadastrados.",
                  response.data.msg,
                  "success"
                );
                self.executaRequest();
                self.checkId = true;
                self.clearReg(); //Chama o método clearReg para limpar o formulário
              }
            })
            .finally(() => (this.loading = false));
        }
      }
    },
    editaReg(userId) {
      const self = this;
      const token =
        "a2279db699464d5026a46688f85e64cca895fe497d8926003f168ef86cd6ed74";
      //Passando o token para uma var
      axios({
        method: "get", //Definindo o método (poderia ser POST, PUT.. ETC)
        url:
          "https://voice.convert.app.br/assist/api/Contato/getFromAssist/?id=" +
          userId,
        //Passando URL da api
        headers: {
          Authorization: "Bearer " + token,
        },

        //O header no axios consiste na passagem de objetos contendo nossos parâmetros definidos
        //de acordo com o que é requisitado para 'x' aplicação
      }).then(function (response) {
        console.log(response.data);
        console.log("Id selected: " + userId);
        self.name = response.data.data.nome;
        self.sobre = response.data.data.sobrenome;
        self.empresa = response.data.data.empresa;
        self.phone = response.data.data.telefoneMovel;
        self.mail = response.data.data.email1;
        self.idType = response.data.data.tipoContato;
        self.isPressed = true; //Pressionou o botão de editar? Então ao cadastrar os dados haverá um condicional para fazer o update invés de create
      });
      this.clickedEdit = true;
      this.userKey = userId;
    },
  },

  // -------------------------------------------------------------------------
  // Carrega os dados da API para fazer o display na tabela
  created() {
    this.executaRequest(); //Chama o método na criação do projeto. Executado somente uma vez
  },
};
</script>

<style scoped>
.table {
  text-align: center;
}
body {
  text-align: center;
}
.container-fluid {
  display: center;
}

.card {
  text-align: center;
  border-radius: 4px;
  margin: 15px;
  padding: 15px;
}
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
.btn {
  margin-top: 10px;
}
.beautify {
  font-family: "Raleway", sans-serif;
}
.searchBar {
  border-radius: 4px;
  margin: 15px;
  padding: 15px;
}
</style>
