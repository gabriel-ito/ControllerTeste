# ControllerTeste

package com.cartoes.api.controllers;

import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.BDDMockito;
import org.mockito.Mockito;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
//import org.springframework.security.test.context.support.WithMockUser;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import com.cartoes.api.dtos.TransacaoDto;
import com.cartoes.api.entities.Transacao;
import com.cartoes.api.services.TransacaoService;
import com.cartoes.api.utils.ConsistenciaException;
import com.cartoes.api.utils.ConversaoUtils;
import com.fasterxml.jackson.databind.ObjectMapper;

@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
@ActiveProfiles("test")
public class TransacaoControllerTest {
	@Autowired
	private MockMvc mvc;
	@MockBean
	private TransacaoService transacaoService;

	private Transacao CriarTransacaoTestes() {
		Transacao transacao = new Transacao();
		transacao.setCnpj("Teste inclusão");
		transacao.setValor(150);
		transacao.setQdtParcelas(1);
		transacao.setJuros(0.1);
		return transacao;
	}

	@Test
	// @WithMockUser
	public void testBuscarPorNumeroCartao() throws Exception {
		Transacao transacao = CriarTransacaoTestes();
		BDDMockito.given(transacaoService.buscarPorNumeroCartao(Mockito.anyString()));
		mvc.perform(MockMvcRequestBuilders.get("/api/transacao/cnpj/5381579886310193").accept(MediaType.APPLICATION_JSON))
				.andExpect(status().isOk()).andExpect(jsonPath("$.dados.id").value(transacao.getId()))
				.andExpect(jsonPath("$.dados.valor").value(transacao.getValor()))
				.andExpect(jsonPath("$.dados.qdtParcelas").value(transacao.getQdtParcelas()))
				.andExpect(jsonPath("$.dados.juros").value(transacao.getJuros()))

				.andExpect(jsonPath("$.erros").isEmpty());
	}

	@Test
	// @WithMockUser
	public void testBuscarPorNumeroCartaoInconsistencia() throws Exception {
		BDDMockito.given(transacaoService.buscarPorNumeroCartao(Mockito.anyString()))
				.willThrow(new ConsistenciaException("Teste inconsistência"));
		mvc.perform(MockMvcRequestBuilders.get("/api/transacao/cnpj/05887098082").accept(MediaType.APPLICATION_JSON))
				.andExpect(status().isBadRequest()).andExpect(jsonPath("$.erros").value("Teste inconsistência"));
	}

	@Test
	// @WithMockUser
	public void testSalvarSucesso() throws Exception {
		Transacao transacao = CriarTransacaoTestes();
		TransacaoDto objEntrada = ConversaoUtils.Converter(transacao);
		String json = new ObjectMapper().writeValueAsString(objEntrada);

		BDDMockito.given(transacaoService.salvar(Mockito.any(Transacao.class))).willReturn(transacao);

		mvc.perform(MockMvcRequestBuilders.post("/api/transacao").content(json).contentType(MediaType.APPLICATION_JSON)
				.accept(MediaType.APPLICATION_JSON)).andExpect(status().isOk())
				.andExpect(jsonPath("$.dados.cnpj").value(objEntrada.getCnpj()))
				.andExpect(jsonPath("$.dados.juros").value(objEntrada.getJuros()))
				.andExpect(jsonPath("$.dados.qdtParcelas").value(objEntrada.getQdtParcelas()))
				.andExpect(jsonPath("$.dados.valor").value(objEntrada.getValor())).andExpect(jsonPath("$.erros").isEmpty());
	}

	@Test
	// @WithMockUser
	public void testSalvarInconsistencia() throws Exception {
		Transacao transacao = CriarTransacaoTestes();
		TransacaoDto objEntrada = ConversaoUtils.Converter(transacao);
		String json = new ObjectMapper().writeValueAsString(objEntrada);
		BDDMockito.given(transacaoService.salvar(Mockito.any(Transacao.class)))
				.willThrow(new ConsistenciaException("Teste inconsistência."));

		mvc.perform(MockMvcRequestBuilders.post("/api/transacao").content(json).contentType(MediaType.APPLICATION_JSON)
				.accept(MediaType.APPLICATION_JSON)).andExpect(status().isBadRequest())
				.andExpect(jsonPath("$.erros").value("Teste inconsistência."));
	}

	@Test
// @WithMockUser
	public void testSalvarCnpjEmBranco() throws Exception {
		TransacaoDto objEntrada = new TransacaoDto();
		objEntrada.setCnpj("");
		objEntrada.setValor("50");
		objEntrada.setQdtParcelas("1");
		objEntrada.setJuros("0");
		String json = new ObjectMapper().writeValueAsString(objEntrada);
		mvc.perform(MockMvcRequestBuilders.post("/api/transacao").content(json).contentType(MediaType.APPLICATION_JSON)
				.accept(MediaType.APPLICATION_JSON)).andExpect(status().isBadRequest())
				.andExpect(jsonPath("$.erros").value("CNPJ não pode ser vazio."));
	}

	@Test
	// @WithMockUser
	public void testSalvarCnpjInvalido() throws Exception {
		TransacaoDto objEntrada = new TransacaoDto();
		objEntrada.setCnpj("1111");
		objEntrada.setValor("50");
		objEntrada.setQdtParcelas("1");
		objEntrada.setJuros("0");
		String json = new ObjectMapper().writeValueAsString(objEntrada);
		mvc.perform(MockMvcRequestBuilders.post("/api/transacao").content(json).contentType(MediaType.APPLICATION_JSON)
				.accept(MediaType.APPLICATION_JSON)).andExpect(status().isBadRequest())
				.andExpect(jsonPath("$.erros").value("CNPJ inválido."));
	}

	@Test
	// @WithMockUser
	public void testSalvarValorEmBranco() throws Exception {
		TransacaoDto objEntrada = new TransacaoDto();
		objEntrada.setCnpj("561981621213212");
		objEntrada.setJuros("1");
		objEntrada.setValor("");
		objEntrada.setQdtParcelas("1");
		String json = new ObjectMapper().writeValueAsString(objEntrada);
		mvc.perform(MockMvcRequestBuilders.post("/api/transacao").content(json).contentType(MediaType.APPLICATION_JSON)
				.accept(MediaType.APPLICATION_JSON)).andExpect(status().isBadRequest())
				.andExpect(jsonPath("$.erros").value("Valor não pode ser vazio."));
	}
	
	public void testSalvarQdtParcelasEmBranco() throws Exception {
		TransacaoDto objEntrada = new TransacaoDto();
		objEntrada.setCnpj("561981621213212");
		objEntrada.setJuros("1");
		objEntrada.setQdtParcelas("");
		String json = new ObjectMapper().writeValueAsString(objEntrada);
		mvc.perform(MockMvcRequestBuilders.post("/api/transacao").content(json).contentType(MediaType.APPLICATION_JSON)
				.accept(MediaType.APPLICATION_JSON)).andExpect(status().isBadRequest())
				.andExpect(jsonPath("$.erros").value("Quantidade Parcelas não pode ser vazio."));
	}
	
	public void testSalvarQdtParcelasInvalido() throws Exception {
		TransacaoDto objEntrada = new TransacaoDto();
		objEntrada.setCnpj("561981621213212");
		objEntrada.setJuros("1");
		objEntrada.setQdtParcelas("3");
		String json = new ObjectMapper().writeValueAsString(objEntrada);
		mvc.perform(MockMvcRequestBuilders.post("/api/transacao").content(json).contentType(MediaType.APPLICATION_JSON)
				.accept(MediaType.APPLICATION_JSON)).andExpect(status().isBadRequest())
				.andExpect(jsonPath("$.erros").value("Quantidade Parcelas deve conter até 2 caracteres numéricos."));
	}
	
	public void testSalvarJurosEmBranco() throws Exception {
		TransacaoDto objEntrada = new TransacaoDto();
		objEntrada.setCnpj("561981621213212");
		objEntrada.setJuros("");
		objEntrada.setValor("500");
		objEntrada.setQdtParcelas("1");
		String json = new ObjectMapper().writeValueAsString(objEntrada);
		mvc.perform(MockMvcRequestBuilders.post("/api/transacao").content(json).contentType(MediaType.APPLICATION_JSON)
				.accept(MediaType.APPLICATION_JSON)).andExpect(status().isBadRequest())
				.andExpect(jsonPath("$.erros").value("Juros não pode ser vazio."));
	}

	@Test
	//	  @WithMockUser
		  public void testSalvarJurosExcedente() throws Exception {
		  TransacaoDto objEntrada = new TransacaoDto();
			objEntrada.setCnpj("561981621213212");
			objEntrada.setJuros("12345");
			objEntrada.setValor("500");
			objEntrada.setQdtParcelas("1");
			String json = new ObjectMapper().writeValueAsString(objEntrada);
			mvc.perform(MockMvcRequestBuilders.post("/api/transacao").content(json).contentType(MediaType.APPLICATION_JSON)
					.accept(MediaType.APPLICATION_JSON)).andExpect(status().isBadRequest())
					.andExpect(jsonPath("$.erros").value("Juros não pode ter mais que 5 caracteres."));
		  }

}
