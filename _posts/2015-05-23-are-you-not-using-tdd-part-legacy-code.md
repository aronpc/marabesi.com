---
layout: post
title: ARE YOU NOT USING TDD?! – PART || (Legacy code)
date: 2015-05-23 18:33:47.000000000 -03:00
image: /images/posts/2015-05-23-are-you-not-using-tdd-part-legacy-code/cover.png
type: post
published: true
status: publish
categories:
- TDD
tags:
- code base,
- features,
- legacy code,
- lines,
- tdd,
- legacy
---

In the part one I introduced the life cycle of TDD
[you can see here]({{ site.baseurl }}{% post_url 2015-04-19-not-using-tdd-part-1 %}) and how it works, but often people come to me and ask about the legacy code how can we start
using TDD with legacy code? Should we start from zero? Should we use baby-steps?

I decided to share my experience in how I used TDD in legacy code and how I did
to get this task done also I have to say many thanks to my friend Alexandre
Cintra who helped my a lot giving me gold tips.

## Edit: Jun 17, 2015

You can find useful information in my slides below. It was created for
[my talk at Samsung Ocean](http://www.meetup.com/pt/THT-Things-Hacker-Team/events/222831499){:target="_blank"}.

<iframe width="100%" height="500" style="border: 1px solid #CCC; border-width: 1px; margin-bottom: 5px; max-width: 100%;" src="//www.slideshare.net/slideshow/embed_code/key/vO17674rb2GiBI" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" allowfullscreen="allowfullscreen"> </iframe>

## Edit: July 28, 2015

Legacy software systems . . . were developed decades ago and have been
continually modified to meet changes in business requirements and computing
platforms. The proliferation of such systems is causing headaches for
large organizations who find them costly to maintain and risky to evolve -
Dayani-Fard

## Legacy code the nightmare of everyone

Doesn't matter if you are in a big company or a small one it is normal to have
legacy code and you will need to maintain and even do some improvements as the
request of our client comes.

How can I identify legacy that is impossible to test?

1. Many responsibilities: Usually legacy code has many lines between 1000 and
2000 (I've seen many with 3000) and do everything in one file database
connection, data persistence, business logic, IO are a couple of examples.

2. Without code pattern: A "great" feature of legacy code is the lack of
pattern in its code. When you touch the code base you usually see huge
files and also blank files, once I was just playing around in a company
repository and I saw a model without one single line just with its declaration
therefore in the controller was everything else the access data, DAO and
even the queries were wrote in the controller!

```php
<?php

class MyLegacyClass {
	
	    // omitted code with 50 lines also construct method and some properties was took off
            public function setXml($xml) {
            	try {
            		$this-&gt;xml = $xml;
            		return $this;
            	} catch(Exception $e) {
            		exit($e-&gt;getMessage());
            	}
            }

            public function getXml() {
            	return $this-&gt;xml;
            }
             
            public function getModalRodoviario() {
            	try {
            		$xml = $this-&gt;getXml();
            		return $xml-&gt;rodo;
            	} catch(Exception $e) {
            		exit($e-&gt;getMessage());
            	}
            }
             
            public function printXML() {
            	$xml = $this-&gt;getXml();
            	print($xml-&gt;asXML());
            	exit;
            }
             
            public function setIdeValues($config, $Cte) {
            	$xml = $this-&gt;getXml();

            	$ide = $xml-&gt;infCte-&gt;addChild('ide');

            	$confIde = $config;

            	$camposObrigatoriosConfIde = array(
			//omitted code 31 lines
            	);

            	$confToma = $confIde['toma'];

            	if($confToma == '4') {
            		$confToma4 = $confIde['toma4'];
            		unset($confIde['toma4']);
            	}

            	unset($confIde['toma']);

            	$confIde['cUF'] = $this-&gt;estadosIBGE[$confIde['cUF']];

            	$this-&gt;chaveAcesso = $this-&gt;calculaChaveAcesso(
            	$confIde['cUF'],
            	substr(str_replace('-', '', $confIde['dhEmi']),2,4),
            	$Cte['Empresa']['Entidade']['ent_cnpj'],
            	$confIde['mod'],
            	$confIde['serie'],
            	$confIde['nCT'],
            	$confIde['tpEmis'],
            	$confIde['cCT']
            	);

            	$Cte-&gt;cte_chave = $this-&gt;chaveAcesso;

            	$Cte-&gt;save();

            	$confIde['cDV'] = $this-&gt;digVer;

            	$xml-&gt;infCte-&gt;addAttribute('Id', 'CTe' . $this-&gt;chaveAcesso);

            	foreach($confIde as $node =&gt; $nodeValue) {
            		if($nodeValue != "" &amp;&amp; !is_array($nodeValue)) {
            			$ide-&gt;addChild($node, $nodeValue);
            		} else {
            			if(array_key_exists($node, $camposObrigatoriosConfIde))
            			throw new Exception($camposObrigatoriosConfIde[$node]);
            		}
            	}

            	//		exit;
            	if(isset($confToma4)) {
            		$toma4 = $ide-&gt;addChild('toma4');
            		$toma4-&gt;addChild('toma', $confToma);
            		 
            		$camposObrigatoriosTomador = array(
				//omitted code 9 lines
            		);
            		 
            		$confToma4['toma_data_ent_cont'] = date('Y-m-d H:i:s');
            		$confToma4['toma_justificativa_cont'] = "Justificativa teste";
            		 
            		foreach($camposObrigatoriosTomador as $campo =&gt; $mensagemErro) {
            			if($confToma4[$campo] == "")
            			throw new Exception($mensagemErro);
            		}
            		 
            		$dePara = array(
				'fone' =&gt; 'toma_telefone',
				'xLgr' =&gt; 'toma_end_logradouro',
				'nro' =&gt; 'toma_end_nro',
				'xCpl' =&gt; 'toma_end_cpl',
				'xBairro' =&gt; 'toma_end_bairo',
				'cMun' =&gt; 'toma_end_cod_mun',
				'xMun' =&gt; 'toma_end_mun',
				'UF' =&gt; 'toma_end_uf_sigla',
            		//'CEP' =&gt; 'toma_end_cep',
				'cPais' =&gt; 'toma_cod_end_pais',
				'xPais' =&gt; 'toma_end_pais',
				'email' =&gt; 'toma_cont_email',
            		//'xJust' =&gt; 'toma_justificativa_cont'
            		);
            		 
            		if(strlen($confToma4['toma_cpf_cnpj']) == 11) {
            			$toma4-&gt;addChild('CPF', $confToma4['toma_cpf_cnpj']);
            			if($confToma4['toma_nome']!="")
            			$toma4-&gt;addChild('xNome', $confToma4['toma_nome']);
            		} else {
            			$toma4-&gt;addChild('CNPJ', $confToma4['toma_cpf_cnpj']);
            			if($confToma4['toma_fantasia']!="")
            			$toma4-&gt;addChild('xFant', $confToma4['toma_fantasia']);
            			if($confToma4['toma_ie']!="")
            			$toma4-&gt;addChild('IE', $confToma4['toma_ie']);
            		}
            		 
            		if(!empty($confToma4['toma_telefone']))
            		$toma4-&gt;addChild('fone', $confToma4['toma_telefone']);
            		 
            		$enderToma = $toma4-&gt;addChild('enderToma');

            		foreach($dePara as $de =&gt; $para) {
            			$confToma4[$para] = $this-&gt;cleaner($confToma4[$para]);
            			if($confToma4[$para]!="")
            			$enderToma-&gt;addChild($de, $confToma4[$para]);
            		}
            		 
            		/*if(!empty($confToma4['toma_data_ent_cont']))
            		 $enderToma-&gt;addChild('dhCont', implode('T', explode(' ', $confToma4['toma_data_ent_cont'])));*/
            	} else {
            		$toma03 = $ide-&gt;addChild('toma03');
            		$toma03-&gt;addChild('toma', $confToma);
            	}

            	$this-&gt;setXml($xml);
            }

            public function setCompl($config) {
            	//		if($this-&gt;validaInputacao($config)) {
            	$xml = $this-&gt;getXml();

            	$compl = $xml-&gt;infCte-&gt;addChild('compl');

            	$confComplPrimeiroBloco = array(
					'xCaracAd', 
					'xCaracSer', 
					'xEmi'
					);

					foreach($confComplPrimeiroBloco as $node) {
						$config[$node] = $this-&gt;cleaner($config[$node]);
						if($config[$node]!="")
						$compl-&gt;addChild($node, $config[$node]);
					}

					if($config['fluxo']['xOrig'] != "" || $config['fluxo']['pass'] != "") {
						$fluxo = $compl-&gt;addChild('fluxo');
					}

					if(!empty($config['fluxo']['xOrig'])) {
						$fluxo-&gt;addChild('xOrig', $config['fluxo']['xOrig']);
					}

					if($config['fluxo']['pass'] != "" &amp;&amp; count($config['fluxo']['pass']) &gt; 0) {
						foreach($config['fluxo']['pass'] as $pass) {
							$xpass = $fluxo-&gt;addChild('pass');

							$nodesPass = array(
						'xPass' =&gt; 'cdp_sigla_aeroporto_passagem',
						'xDest' =&gt; 'cdp_sigla_aeroporto_destino',
						'xRota' =&gt; 'cdp_codigo_rota_entrega'
						);
							
						foreach($nodesPass as $nodePass =&gt; $nodePassIndex) {
							$pass[$nodePassIndex] = $this-&gt;cleaner($pass[$nodePassIndex]);
							if($pass[$nodePassIndex]!="")
							$xpass-&gt;addChild($nodePass, $pass[$nodePassIndex]);
						}
						}
					}

					if(count($config['Entrega'])) {
						$Entrega = $compl-&gt;addChild('Entrega');

						$config['Entrega']['tpPer'] = $this-&gt;cleaner($config['Entrega']['tpPer']);

						if($config['Entrega']['tpPer']=="")

						switch($config['Entrega']['tpPer']) {
							case '0' :
								$semData = $Entrega-&gt;addChild('semData');
								$semData-&gt;addChild('tpPer', $config['Entrega']['tpPer']);
								break;
							case '1' :
							case '2' :
							case '3' :
								$comData = $Entrega-&gt;addChild('comData');
								$comData-&gt;addChild('tpPer', $config['Entrega']['tpPer']);
								$config['Entrega']['dProg'] = $this-&gt;cleaner($config['Entrega']['dProg']);

								if($config['Entrega']['dProg']!="")
								$comData-&gt;addChild('tpPer', $config['Entrega']['dProg']);
								
								break;
							case '4' :
								$noPeriodo = $Entrega-&gt;addChild('noPeriodo');
								$noPeriodo-&gt;addChild('tpPer', $config['Entrega']['tpPer']);
								$config['Entrega']['dtIni'] = $this-&gt;cleaner($config['Entrega']['dtIni']);
								if($config['Entrega']['dtIni']!="")
								$noPeriodo-&gt;addChild('dtIni', $config['Entrega']['dtIni']);
								
								$config['Entrega']['dtFim'] = $this-&gt;cleaner($config['Entrega']['dtFim']);
								if($config['Entrega']['dtFim']!="")
								$noPeriodo-&gt;addChild('dtFim', $config['Entrega']['dtFim']);
								
								break;
						}

						$config['Entrega']['tpHor'] = $this-&gt;cleaner($config['Entrega']['tpHor']);

						switch($config['Entrega']['tpHor']) {
							case '0' :
								$semHora = $Entrega-&gt;addChild('semHora');
								$semHora-&gt;addChild('tpHor', $config['Entrega']['tpHor']);
								break;
							case '1' :
							case '2' :
							case '3' :
								$comHora = $Entrega-&gt;addChild('comHora');
								$comHora-&gt;addChild('tpHor', $config['Entrega']['tpHor']);
								$config['Entrega']['hProg'] = $this-&gt;cleaner($config['Entrega']['hProg']);
								if($config['Entrega']['hProg']!="")
								$comHora-&gt;addChild('tpHor', $config['Entrega']['hProg']);
								break;
							case '4' :
								$noInter = $Entrega-&gt;addChild('noInter');
								$noInter-&gt;addChild('tpHor', $config['Entrega']['tpHor']);
								$config['Entrega']['hIni'] = $this-&gt;cleaner($config['Entrega']['hIni']);
								if($config['Entrega']['hIni']!="")
								$noInter-&gt;addChild('hIni', $config['Entrega']['hIni']);
								$config['Entrega']['hFim'] = $this-&gt;cleaner($config['Entrega']['hFim']);
								if($config['Entrega']['hFim']!="")
								$noInter-&gt;addChild('hFim', $config['Entrega']['hFim']);
								break;
						}
					}

					if ( !empty($config['xObs']) ){
						$compl-&gt;addChild('xObs', $config['xObs']);
					}

					if(isset($config['ObsCont']) &amp;&amp; count($config['ObsCont']) &gt; 0) {
						foreach($config['ObsCont'] as $ObsCont) {
							$xObsCont = $compl-&gt;addChild('ObsCont');

							$ObsCont['coic_identificador'] = $this-&gt;cleaner($ObsCont['coic_identificador']);
							if($ObsCont['coic_identificador']!="")
							$xObsCont-&gt;addChild('xCampo', $ObsCont['coic_identificador']);
							$ObsCont['coic_obs'] = $this-&gt;cleaner($ObsCont['coic_obs']);
							if($ObsCont['coic_obs']!="")
							$xObsCont-&gt;addChild('xTexto', $ObsCont['coic_obs']);
						}
					}

					if(isset($config['ObsFisco']) &amp;&amp; count($config['ObsFisco']) &gt; 0) {
						foreach($config['ObsFisco'] as $ObsFisco) {
							$xObsFisco = $compl-&gt;addChild('ObsFisco');

							$ObsFisco['coif_identificador'] = $this-&gt;cleaner($ObsFisco['coif_identificador']);
							if($ObsFisco['coif_identificador']!="")
							$xObsFisco-&gt;addChild('xCampo', $ObsFisco['coif_identificador']);
							$ObsFisco['coif_obs'] = $this-&gt;cleaner($ObsFisco['coif_obs']);
							if($ObsFisco['coif_obs']!="")
							$xObsFisco-&gt;addChild('xTexto', $ObsFisco['coif_obs']);
						}
					}

					$this-&gt;setXml($xml);
					//		}
            }

            public function setEmitValues($config) {
            	$xml = $this-&gt;getXml();

            	$emitDePara = array(
			'CNPJ' =&gt; 'CNPJ',
			'IE' =&gt; 'IE',
			'xNome' =&gt; 'nome_razao',
			'xFant' =&gt; 'fantasia'
			);

			$camposObrigatorios = array(
			//omitted code 3 lines
			);

			$emit = $xml-&gt;infCte-&gt;addChild('emit');

			foreach($emitDePara as $node =&gt; $nodeValue) {
				$config[$nodeValue] = $this-&gt;cleaner($config[$nodeValue]);
					
				if($config[$nodeValue]!="") {
					$emit-&gt;addChild($node, $config[$nodeValue]);
				} else {
					if(array_key_exists($node, $camposObrigatorios)) {
						throw new Exception($camposObrigatorios[$node]);
					}
				}
			}

			$emitEnderDePara = array(
				'xLgr' =&gt; 'logradouro',
				'nro' =&gt; 'numero',
				'xCpl' =&gt; 'complemento',
				'xBairro' =&gt; 'bairro',
				'cMun' =&gt; 'cod_municipio',
				'xMun' =&gt; 'municipio',
				'UF' =&gt; 'UF',
				'fone' =&gt; 'fone'
				);
					
				$camposObrigatoriosEnder = array(
				//omitted code 7 lines
				);
					
				$enderEmit = $emit-&gt;addChild('enderEmit');
					
				foreach($emitEnderDePara as $node =&gt; $nodeValue) {
					$config[$nodeValue] = $this-&gt;cleaner($config[$nodeValue]);

					if($config[$nodeValue]!="") {
						$enderEmit-&gt;addChild($node, $config[$nodeValue]);
					} else {
						if(array_key_exists($node, $camposObrigatoriosEnder)) {
							throw new Exception($camposObrigatoriosEnder[$node]);
						}
					}
				}
					
				$this-&gt;setXml($xml);
            }

            public function setRemValues($config) {
            	$config = $this-&gt;limpaEspacos($config);
            	$xml = $this-&gt;getXml();

            	$rem = $xml-&gt;infCte-&gt;addChild('rem');
            	if(strlen($config['cpfcnpj']) == 14) {
            		$rem-&gt;addChild('CNPJ', str_pad($config['cpfcnpj'], 14, '0', STR_PAD_LEFT));
            	} else {
            		$rem-&gt;addChild('CPF', str_pad($config['cpfcnpj'], 11, '0', STR_PAD_LEFT));
            	}

            	if ( !is_numeric($config['IE']) ) {
            		$rem-&gt;addChild('IE', '000000000');
            	} else {
            		$rem-&gt;addChild('IE', str_pad($config['IE'], 9, STR_PAD_LEFT));
            	}

            	if ( TIPO_AMBIENTE_CTE == 2 ) {
            		$config['nome'] = 'CT-E EMITIDO EM AMBIENTE DE HOMOLOGACAO - SEM VALOR FISCAL';
            		$config['xFant'] = 'CT-E EMITIDO EM AMBIENTE DE HOMOLOGACAO - SEM VALOR FISCAL';
            	}

            	$rem-&gt;addChild('xNome',$config['nome']);
            	$rem-&gt;addChild('xFant', $config['nome']);

            	if (!empty($config['fone'])){
            		$rem-&gt;addChild('fone', $config['fone']);
            	}
            	$enderReme = $rem-&gt;addChild('enderReme');

            	$enderReme-&gt;addChild('xLgr', $config['logradouro']);
            	$enderReme-&gt;addChild('nro', $config['numero']);
            	$enderReme-&gt;addChild('xCpl', $config['bairro']);
            	$enderReme-&gt;addChild('xBairro', $config['bairro']);
            	$enderReme-&gt;addChild('cMun', $config['cod_municipio']);
            	$enderReme-&gt;addChild('xMun', $config['municipio']);

            	if ( !empty($config['CEP']) ) {
            		$enderReme-&gt;addChild('CEP', str_pad($config['CEP'], 8, STR_PAD_LEFT));
            	}

            	$enderReme-&gt;addChild('UF', $config['UF']);
            	$enderReme-&gt;addChild('cPais', $config['cod_pais']);
            	$enderReme-&gt;addChild('xPais', $config['pais']);

            	if ( !empty($config['email']) ) {
            		$rem-&gt;addChild('email', $config['email']);
            	}

            	
            	$this-&gt;setXml($xml);

            }
             
            public function setDestValues($config) {
            	$xml = $this-&gt;getXml();

            	$dest = $xml-&gt;infCte-&gt;addChild('dest');

            	if(strlen($config['cpfcnpj']) == 14) {
            		$dest-&gt;addChild('CNPJ', $config['cpfcnpj']);
            	} else {
            		$dest-&gt;addChild('CPF', $config['cpfcnpj']);
            	}

            	if ( !is_numeric($config['IE']) ) {
            		$dest-&gt;addChild('IE', '000000000');
            	} else {
            		$dest-&gt;addChild('IE', str_pad($config['IE'], 9, STR_PAD_LEFT));
            	}

            	if ( TIPO_AMBIENTE_CTE == 2 ) {
            		$config['nome'] = 'CT-E EMITIDO EM AMBIENTE DE HOMOLOGACAO - SEM VALOR FISCAL';
            	}

            	$dest-&gt;addChild('xNome', $config['nome']);

            	if( !empty($config['fone']) ) {
            		$dest-&gt;addChild('fone', $config['fone']);
            	}

            	$enderDest = $dest-&gt;addChild('enderDest');
            	$enderDest-&gt;addChild('xLgr', $config['logradouro']);
            	$enderDest-&gt;addChild('nro', str_pad($config['numero'], 3, STR_PAD_LEFT));

            	if( !empty($config['complemento']) ) {
            		$enderDest-&gt;addChild('xCpl', $config['complemento']);
            	}
            	$enderDest-&gt;addChild('xBairro', $config['bairro']);
            	$enderDest-&gt;addChild('cMun', $config['cod_municipio']);
            	$enderDest-&gt;addChild('xMun', $config['municipio']);

            	if( !empty($config['CEP']) ) {
            		$enderDest-&gt;addChild('CEP', str_pad($this-&gt;cleaner($config['CEP']), 8, '0', STR_PAD_LEFT));
            	}

            	$enderDest-&gt;addChild('UF', $config['UF']);

            	if( !empty($config['cod_pais']) ) {
            		$enderDest-&gt;addChild('cPais', $config['cod_pais']);
            	}
            	if( !empty($config['pais']) ) {
            		$enderDest-&gt;addChild('xPais', $config['pais']);
            	}

            	if( !empty($config['email']) ) {
            		$dest-&gt;addChild('email', $config['email']);
            	}

            

            	$this-&gt;setXml($xml);
            }
             
            public function setRecValues($config) {
            	$xml = $this-&gt;getXml();

            	$rec = $xml-&gt;infCte-&gt;addChild('receb');

            	if(strlen($config['cpfcnpj']) == 14) {
            		$rec-&gt;addChild('CNPJ', $config['cpfcnpj']);
            	} else {
            		$rec-&gt;addChild('CPF', $config['cpfcnpj']);
            	}

            	if ( !is_numeric($config['IE']) ) {
            		$rec-&gt;addChild('IE', '000000000');
            	} else {
            		$rec-&gt;addChild('IE', str_pad($config['IE'], 9, STR_PAD_LEFT));
            	}

            	if ( TIPO_AMBIENTE_CTE == 2 ) {
            		$config['nome'] = 'CT-E EMITIDO EM AMBIENTE DE HOMOLOGACAO - SEM VALOR FISCAL';
            	}
            	$rec-&gt;addChild('xNome', $config['nome']);

            	if( !empty($config['fone']) ) {
            		$rec-&gt;addChild('fone', $config['fone']);
            	}
            	$enderRec = $rec-&gt;addChild('enderReceb');
            	$enderRec-&gt;addChild('xLgr', $config['logradouro']);
            	$enderRec-&gt;addChild('nro', str_pad($config['numero'], 3, STR_PAD_LEFT));

            	if( !empty($config['complemento']) ) {
            		$enderRec-&gt;addChild('xCpl', $config['complemento']);
            	}
            	$enderRec-&gt;addChild('xBairro', $config['bairro']);
            	$enderRec-&gt;addChild('cMun', $config['cod_municipio']);
            	$enderRec-&gt;addChild('xMun', $config['municipio']);

            	if( !empty($config['CEP']) ) {
            		$enderRec-&gt;addChild('CEP', $config['CEP']);
            	}

            	$enderRec-&gt;addChild('UF', $config['UF']);

            	if( !empty($config['cod_pais']) ){
            		$enderRec-&gt;addChild('cPais', $config['cod_pais']);
            	}

            	if( empty($config['pais']) ) {
            		$enderRec-&gt;addChild('xPais', $config['pais']);
            	}

            	if( !empty($config['email']) ) {
            		$rec-&gt;addChild('email', $config['email']);
            	}
            	$this-&gt;setXml($xml);
            }

            public function setExpValues($config) {
            	$xml = $this-&gt;getXml();
            	$exp = $xml-&gt;infCte-&gt;addChild('exped');

            	if(strlen($config['cpfcnpj']) == 14) {
            		$exp-&gt;addChild('CNPJ', $config['cpfcnpj']);
            	} else {
            		$exp-&gt;addChild('CPF', $config['cpfcnpj']);
            	}

            	if ( !is_numeric($config['IE']) ) {
            		$exp-&gt;addChild('IE', '000000000');
            	} else {
            		$exp-&gt;addChild('IE', str_pad($config['IE'], 9, STR_PAD_LEFT));
            	}

            	if ( TIPO_AMBIENTE_CTE == 2 ) {
            		$config['nome'] = 'CT-E EMITIDO EM AMBIENTE DE HOMOLOGACAO - SEM VALOR FISCAL';
            	}
            	$exp-&gt;addChild('xNome', $config['nome']);

            	$config['fone'] = $this-&gt;cleaner($config['fone']);

            	if( !empty($config['fone']) ) {
            		$exp-&gt;addChild('fone', $config['fone']);
            	}

            	$enderExp = $exp-&gt;addChild('enderExped');
            	$enderExp-&gt;addChild('xLgr', $config['logradouro']);
            	$enderExp-&gt;addChild('nro', str_pad($config['numero'], 3, STR_PAD_LEFT));

            	if($this-&gt;cleaner($config['complemento']))
            	$enderExp-&gt;addChild('xCpl', $config['complemento']);

            	$enderExp-&gt;addChild('xBairro', $config['bairro']);
            	$enderExp-&gt;addChild('cMun', $config['cod_municipio']);
            	$enderExp-&gt;addChild('xMun', $config['municipio']);

            	if($this-&gt;cleaner($config['CEP']))
            	$enderExp-&gt;addChild('CEP', $config['CEP']);

            	$enderExp-&gt;addChild('UF', $config['UF']);

            	if($this-&gt;cleaner($config['cod_pais'])!="")
            	$enderExp-&gt;addChild('cPais', $config['cod_pais']);

            	if($this-&gt;cleaner($config['pais'])!="")
            	$enderExp-&gt;addChild('xPais', $config['pais']);

            	if( !empty($config['email']) ) {
            		$exp-&gt;addChild('email', $config['email']);
            	}

            	$this-&gt;setXml($xml);
            }

            
            public function setInfCteComp($config) {
            	$xml = $this-&gt;getXml();

            	if($config['chave']!="") {
            		$infCteComp = $xml-&gt;infCte-&gt;addChild('infCteComp');
            		$infCteComp-&gt;addChild('chave', $config['chave']);
            	}

            	$this-&gt;setXml($xml);
            }

            public function setInfCteAnu($config) {
            	$xml = $this-&gt;getXml();

            	if($config['chCte']!="" &amp;&amp; $config['dEmi']!="") {
            		$infCteAnu = $xml-&gt;infCte-&gt;addChild('infCteAnu');
            		$infCteAnu-&gt;addChild('chCte', $config['chCte']);
            		$infCteAnu-&gt;addChild('dEmi', $config['dEmi']);
            	}

            	$this-&gt;setXml($xml);
            }

            public function setAutXML($config) {
            	$xml = $this-&gt;getXml();

            	foreach($config as $cda) {
            		$autXML = $xml-&gt;infCte-&gt;addChild('autXML');
            		 
            		if(strlen($cda['cda_cpf_cnpj_autori']) == 14)
            		$autXML-&gt;addChild('CNPJ', $cda['cda_cpf_cnpj_autori']);
            		else
            		$autXML-&gt;addChild('CPF', $cda['cda_cpf_cnpj_autori']);
            	}

            	$this-&gt;setXml($xml);
            }

            public function setModalRodoviario($config) {
            	$xml = $this-&gt;getXml();
            	$rodo = $xml-&gt;infCte-&gt;infCTeNorm-&gt;infModal-&gt;addChild('rodo');
            	$modal = $config['rodo'];

            	$rodo-&gt;addChild('RNTRC', $modal['RNTRC']);
            	$rodo-&gt;addChild('dPrev', $modal['dPrev']);
            	$rodo-&gt;addChild('lota', $modal['lota']);

            	if( !empty($modal['CIOT']) ) {
            		$rodo-&gt;addChild('CIOT', str_pad($this-&gt;cleaner($modal['CIOT']), 12, STR_PAD_LEFT));
            	}

            	if(isset($modal['occ']) &amp;&amp; count($modal['occ']) &gt; 0) {
            		foreach($modal['occ'] as $occ) {
            			$xocc = $rodo-&gt;addChild('occ');

            			$xocc-&gt;addChild('serie', $occ['occ_serie']);
            			$xocc-&gt;addChild('nOcc', $occ['occ_numero']);
            			$xocc-&gt;addChild('dEmi', $occ['occ_demi']);

            			$emiOcc = $xocc-&gt;addChild('emiOcc');

            			$emiOcc-&gt;addChild('CNPJ', $occ['occ_cnpj']);

            			if($this-&gt;cleaner($occ['occ_cint'])!="")
            			$emiOcc-&gt;addChild('cInt', $occ['occ_cint']);

            			$emiOcc-&gt;addChild('IE', $occ['occ_ie']);
            			$emiOcc-&gt;addChild('UF', $occ['Uf']['uf_sigla']);

            			if($this-&gt;cleaner($occ['occ_fone'])!="")
            			$emiOcc-&gt;addChild('fone', $occ['occ_fone']);
            		}
            	}

            	if(isset($modal['valePed']) &amp;&amp; count($modal['valePed']) &gt; 0) {
            		foreach($modal['valePed'] as $valePed) {
            			$xValePed = $rodo-&gt;addChild('valePed');

            			$xValePed-&gt;addChild('CNPJForn', $valePed['vpd_cnpj_forn']);
            			$xValePed-&gt;addChild('nCompra', $valePed['vpd_ncompra']);

            			if($this-&gt;cleaner($valePed['vpd_cnpj_pg'])!="")
            			$xValePed-&gt;addChild('CNPJPg', $valePed['vpd_cnpj_pg']);

            			$xValePed-&gt;addChild('vValePed', $valePed['vpd_valor_vale_pedagio']);
            		}
            	}

            	if(isset($modal['veic']) &amp;&amp; count($modal['veic']) &gt; 0) {
            		foreach($modal['veic'] as $confVeic){
            			$veic = $rodo-&gt;addChild('veic');

            			if($this-&gt;cleaner($confVeic['CteVeiculo']['cve_cod_interno'])!="")
            			$veic-&gt;addChild('cInt', $confVeic['CteVeiculo']['cve_cod_interno']);

            			$veic-&gt;addChild('RENAVAM', $confVeic['CteVeiculo']['cve_renavam']);
            			$veic-&gt;addChild('placa', str_replace(' ', '', $confVeic['CteVeiculo']['cve_placa'])); // remove qualquer espa\E7o branco
            			$veic-&gt;addChild('tara', $confVeic['CteVeiculo']['cve_tara']);
            			$veic-&gt;addChild('capKG', $confVeic['CteVeiculo']['cve_capacidade_kg']);
            			$veic-&gt;addChild('capM3', $confVeic['CteVeiculo']['cve_capacidade_mc']);
            			$veic-&gt;addChild('tpProp', $confVeic['CteVeiculo']['cve_proprietario']);
            			$veic-&gt;addChild('tpVeic', $confVeic['CteVeiculo']['CteVeiculoTipo']['cvt_codigo']);
            			$veic-&gt;addChild('tpRod', $confVeic['CteVeiculo']['CteVeiculoRodado']['cvr_codigo']);
            			$veic-&gt;addChild('tpCar', $confVeic['CteVeiculo']['CteVeiculoCarroceria']['cvc_codigo']);
            			$veic-&gt;addChild('UF', $confVeic['CteVeiculo']['Uf']['uf_sigla']);

            			if($confVeic['CteVeiculo']['cve_proprietario'] != 'P') {
            				$prop = $rodo-&gt;addChild('prop');
            				$infoVeicProp = CteVeiculo::consultaVeiculo($confVeic['CteVeiculo']['cve_id']);
            				 
            				$veicProp = reset($infoVeicProp['CteEmpresaVeiculoProprietarioAtrb']);
            				$veicPropTipo = $veicProp['CteEmpresaVeiculoProprietarioTipo'];
            				 
            				if($veicProp['evpa_tipo_pessoa'] == 'FIS'){
            					$prop-&gt;addChild('CPF', $veicProp['evpa_cpf_cnpj']);
            				} else {
            					$prop-&gt;addChild('CNPJ', $veicProp['evpa_cpf_cnpj']);
            				}
            				$prop-&gt;addChild('RNTRC', $veicProp['evpa_rntrc']);
            				$prop-&gt;addChild('xNome', $veicProp['evpa_nome_razao']);
            				$prop-&gt;addChild('IE', $veicProp['evpa_rg_ie']);
            				$prop-&gt;addChild('UF', $veicProp['Uf']['uf_sigla']);
            				$prop-&gt;addChild('tpProp', $veicPropTipo['evpt_codigo']);
            			}
            		}
            	}

            	if(isset($modal['lacre']) &amp;&amp; count($modal['lacre']) &gt; 0) {
            		foreach($modal['lacre'] as $infLacre) {
            			$lacRodo = $rodo-&gt;addChild('lacRodo');

            			$lacRodo-&gt;addChild('nLacre', $infLacre['cmrl_num_lacre']);
            		}
            	}

            	if(isset($modal['moto']) &amp;&amp; count($modal['moto']) &gt; 0) {
            		foreach($modal['moto'] as $infMoto) {
            			$moto = $rodo-&gt;addChild('moto');

            			$moto-&gt;addChild('xNome', $infMoto['CteMotorista']['mot_nome']);
            			$moto-&gt;addChild('CPF', $infMoto['CteMotorista']['mot_cpf']);
            		}
            	}

            	$this-&gt;setXml($xml);
            }

            public function setVPrestValues($config) {
            	$xml = $this-&gt;getXml();

            	$vPrest = $xml-&gt;infCte-&gt;addChild('vPrest');

            	$vPrest-&gt;addChild('vTPrest', empty($config['vTPrest'])? '0.00' : $config['vTPrest']);
            	$vPrest-&gt;addChild('vRec', empty($config['vRec'])? '0.00' : $config['vRec']);

            	if(isset($config['Comp']) &amp;&amp; count($config['Comp']) &gt; 0) {
            		// \E8 obrigat\F3rio ter um elemento filho do Comp para emitir a nota
            		foreach($config['Comp'] as $Comp) {
            			// \E8 necess\E1rio fazer a verifica\E7\E3o para n\E3o ocorrer erros no xml
            			if ( !empty($Comp['cic_nome_componente']) &amp;&amp; !empty($Comp['cic_valor_componente']) ){
            				$xComp = $vPrest-&gt;addChild('Comp');
            				// verificar se esse campo \E9 vazio caso contr\E1rio ocorrer\E1 um erro no xml
            				$xComp-&gt;addChild('xNome', (empty($Comp['cic_nome_componente']))? 'ND' : $Comp['cic_nome_componente']);
            				$xComp-&gt;addChild('vComp', (empty($Comp['cic_valor_componente']))? '00.00' : $Comp['cic_valor_componente']); //13,2 ER25 15 posi\E7\F5es, sendo 13 inteiras e 2 decimais.
            			}
            		}
            	}

            	$this-&gt;setXml($xml);
            }

            public function setImpValues($config) {
            	$xml = $this-&gt;getXml();

            	$imp = $xml-&gt;infCte-&gt;addChild('imp');

            	$ICMS = $imp-&gt;addChild('ICMS');

            	switch($config['CST']) {
            		case '00' :
            			$ICMS00 = $ICMS-&gt;addChild('ICMS00');

            			$ICMS00-&gt;addChild('CST', $config['CST']);
            			$ICMS00-&gt;addChild('vBC', $config['vBC']);
            			$ICMS00-&gt;addChild('pICMS', $config['pICMS']);
            			$ICMS00-&gt;addChild('vICMS', $config['vICMS']);
            			break;
            		case '20' :
            			$ICMS20 = $ICMS-&gt;addChild('ICMS20');
            			$ICMS20-&gt;addChild('CST', $config['CST']);
            			$ICMS20-&gt;addChild('pRedBC', $config['pRedBC']);
            			$ICMS20-&gt;addChild('vBC', $config['vBC']);
            			$ICMS20-&gt;addChild('pICMS', $config['pICMS']);
            			$ICMS20-&gt;addChild('vICMS', $config['vICMS']);
            			break;

            		case '40' :
            		case '41' :
            		case '51' :
            			$ICMS45 = $ICMS-&gt;addChild('ICMS45');
            			$ICMS45-&gt;addChild('CST', $config['CST']);
            			break;

            		case '60' :
            			$ICMS60 = $ICMS-&gt;addChild('ICMS60');

            			$ICMS60-&gt;addChild('CST', $config['CST']);
            			$ICMS60-&gt;addChild('vBCSTRet', $config['vBCSTRet']);
            			$ICMS60-&gt;addChild('vICMSSTRet', $config['vICMSSTRet']);
            			$ICMS60-&gt;addChild('pICMSSTRet', $config['pICMSSTRet']);
            			$ICMS60-&gt;addChild('vCred', $config['vCred']);
            			break;

            		case '90' :
            			$ICMS90 = $ICMS-&gt;addChild('ICMS90');
            			$ICMS90-&gt;addChild('CST', $config['CST']);
            			$ICMS90-&gt;addChild('pRedBC', $config['pRedBC']);
            			$ICMS90-&gt;addChild('vBC', $config['vBC']);
            			$ICMS90-&gt;addChild('pICMS', $config['pICMS']);
            			$ICMS90-&gt;addChild('vICMS', $config['Vicms']);
            			$ICMS90-&gt;addChild('vCred', $config['vCred']);
            			break;

            		case 'SN' :
            			$ICMSSN = $ICMS-&gt;addChild('ICMSSN');
            			$ICMSSN-&gt;addChild('indSN', 1);
            			break;

            		default: // Default ser\E1 outra UF
            			$ICMSOutraUF = $ICMS-&gt;addChild('ICMSOutraUF');

            			$ICMSOutraUF-&gt;addChild('CST', '90');
            			$ICMSOutraUF-&gt;addChild('pRedBCOutraUF', $config['pRedBCOutraUF']);
            			$ICMSOutraUF-&gt;addChild('vBCOutraUF', $config['vBCOutraUF']);
            			$ICMSOutraUF-&gt;addChild('pICMSOutraUF', $config['pICMSOutraUF']);
            			$ICMSOutraUF-&gt;addChild('vICMSOutraUF', $config['vICMSOutraUF']);
            	}

            	if ( !empty($config['vTotTrib']) ) {
            		$imp-&gt;addChild('vTotTrib', $config['vTotTrib']);
            	}

            	if ( !empty($config['infAdFisco']) ) {
            		$imp-&gt;addChild('infAdFisco', $config['infAdFisco']);
            	}

            	$this-&gt;setXml($xml);
            }

            public function setInfCTeNormValues($config) {
            	$xml = $this-&gt;getXml();

            	$infCTeNorm = $xml-&gt;infCte-&gt;addChild('infCTeNorm');
            	$dInfCarga = $config['infCarga'];

            	$infCarga = $infCTeNorm-&gt;addChild('infCarga');

            	if( $this-&gt;cleaner( $dInfCarga['vCarga']) != '' ){
            		$infCarga-&gt;addChild('vCarga', $dInfCarga['vCarga']);
            	}

            	if ( empty($dInfCarga['proPred']) ){
            		$dInfCarga['proPred'] = 'N/D';
            	}
            	$infCarga-&gt;addChild('proPred', $dInfCarga['proPred']);

            	if( $this-&gt;cleaner($dInfCarga['xOutCat']) != '') {
            		$infCarga-&gt;addChild('xOutCat', $dInfCarga['xOutCat']);
            	}

            	if(count($config['infQ']) &gt; 0) {
            		foreach($config['infQ'] as $dInfQ) {
            			$infQ = $infCarga-&gt;addChild('infQ');

            			$infQ-&gt;addChild('cUnid', str_pad($dInfQ['cnc_codigo_unidade_medida'], 2, '0', STR_PAD_LEFT)/*'01'*/);
            			$infQ-&gt;addChild('tpMed', $dInfQ['cnc_tipo_medida']/*'PESO BRUTO'*/);
            			$infQ-&gt;addChild('qCarga', $dInfQ['cnc_valor_componente']/*'12345678911.1154'*/);
            		}
            	}

  
            	$documentos = $config['infDoc'];
            	if ( isset($documentos) &amp;&amp; count($documentos) &gt; 0){

            		$infDoc = $infCTeNorm-&gt;addChild('infDoc');
            		foreach($documentos as $dInfDoc) {
            			
            			switch($dInfDoc['cnf_tipo']) {
            				case 1 :
            					$infNF = $infDoc-&gt;addChild('infNF');
            					$infNF-&gt;addChild('nRoma', str_pad($dInfDoc['cnf_numero_romaneio'], 20, STR_PAD_LEFT)/*'12315646156189189111'*/);
            					$infNF-&gt;addChild('nPed', str_pad($dInfDoc['cnf_numero_pedido'], 20, STR_PAD_LEFT)/*'12315646156189189111'*/);
            					$infNF-&gt;addChild('mod', '01'/*$dInfDoc['cnf_modelo_nota']'04'*/);	//Preencher com:01 - NF Modelo 01/1A e Avulsa;04 - NF de Produtor
            					$infNF-&gt;addChild('serie', str_pad($dInfDoc['cnf_serie'], 3, STR_PAD_LEFT)/*'1'*/);

            					$infNF-&gt;addChild('nDoc', str_pad($dInfDoc['cnf_numero'], 20, STR_PAD_LEFT)/*'1'*/);

            					$infNF-&gt;addChild('dEmi', ($dInfDoc['cnf_data_emissao'] == '---')? $dInfDoc['cnf_data_emissao'] : date('Y-m-d')); /
            					$infNF-&gt;addChild('vBC', ($dInfDoc['cnf_valor_base_icms'])? $dInfDoc['cnf_valor_base_icms'] : '0.00'/*'8047.11'*/);
            					$infNF-&gt;addChild('vICMS', ($dInfDoc['cnf_total_icms'])? $dInfDoc['cnf_total_icms'] : '0.00'/*'965.65'*/);
            					$infNF-&gt;addChild('vBCST', ($dInfDoc['cnf_valor_base_icms_st'])? $dInfDoc['cnf_valor_base_icms_st'] : '0.00'/*'123.00'*/);
            					$infNF-&gt;addChild('vST', ($dInfDoc['cnf_valor_icms_st'])? $dInfDoc['cnf_valor_icms_st'] : '0.00'/*'123.00'*/);
            					$infNF-&gt;addChild('vProd', ($dInfDoc['cnf_valor_total_produtos'])? $dInfDoc['cnf_valor_total_produtos'] : '0.00'/*'123.00'*/);
            					$infNF-&gt;addChild('vNF', ($dInfDoc['cnf_valor_total_nf'])? $dInfDoc['cnf_valor_total_nf'] : '0.00'/*'123.00'*/);
            					$infNF-&gt;addChild('nCFOP', ($dInfDoc['cnf_cfop_prodominante'])? $dInfDoc['cnf_cfop_prodominante'] : '5353'/*'5353'*/);

            					if ( !empty($dInfDoc['cnf_peso_total']) ){
            						$infNF-&gt;addChild('nPeso', $dInfDoc['cnf_peso_total']);
            					}

            					if ( !empty($dInfDoc['cnf_suframa']) ) {
            						$infNF-&gt;addChild('PIN', str_pad($dInfDoc['cnf_suframa'], 9, STR_PAD_LEFT/*'123'*/));
            					}

            					if ( !empty($dInfDoc['cnf_data_prevista_entrega']) &amp;&amp; $dInfDoc['cnf_data_prevista_entrega'] != '---' ) { 
            						$infNF-&gt;addChild('dPrev', $dInfDoc['cnf_data_prevista_entrega']);
            					}
            					break;
            				case 2 :
            					$infNFe = $infDoc-&gt;addChild('infNFe');
            					$infNFe-&gt;addChild('chave', $dInfDoc['cnf_eletronica_chave_acesso']);

            					if ( !empty($dInfDoc['cnf_eletronica_suframa']) ){
            						$infNFe-&gt;addChild('PIN', str_pad($dInfDoc['cnf_eletronica_suframa'], 9, STR_PAD_LEFT));
            					}

            					if ( !empty($dInfDoc['cnf_eletronica_data_prevista']) &amp;&amp; $dInfDoc['cnf_eletronica_data_prevista'] != '---' ){ 
            						$infNFe-&gt;addChild('dPrev', $dInfDoc['cnf_eletronica_data_prevista']);
            					}

            					$unidadeTransporteNFe = $dInfDoc['CteInfNotaFiscalUnidadeTransporte'];
            					if(count($unidadeTransporteNFe) &gt; 0) {
            						foreach($unidadeTransporteNFe as $dInfUnidTranspNFe) {
            							$infUnidTranspNFe = $infNFe-&gt;addChild('infUnidTransp');

            							$infUnidTranspNFe-&gt;addChild('tpUnidTransp', $dInfUnidTranspNFe['cut_tipo_unidade_transportadora']);
            							$infUnidTranspNFe-&gt;addChild('idUnidTransp', $dInfUnidTranspNFe['cut_identificacao_unidade_transportadora']);

            							$lacresUnidTranspNFe = $dInfUnidTranspNFe['CteInfNotaFiscalUnidadeTransporteLacre'];

            							if(count($lacresUnidTranspNFe) &gt; 0) {
            								foreach($lacresUnidTranspNFe as $dLacUnidTranspNFe) {
            									$lacUnidTranspNFe = $infUnidTranspNFe-&gt;addChild('lacUnidTransp');

            									$lacUnidTranspNFe-&gt;addChild('nLacre', $dLacUnidTranspNFe['ctl_numero_lacre']);
            								}
            							}

            							$unidadesDeCargaTransNFe = $dInfUnidTranspNFe['CteInfNotaFiscalUnidadeCarga'];

            							if(count($unidadesDeCargaTransNFe) &gt; 0) {
            								foreach($unidadesDeCargaTransNFe as $dInfUnidCargaTranspNFe) {
            									$infUnidCargaTranspNFe = $infUnidTranspNFe-&gt;addChild('infUnidCarga');

            									$infUnidCargaTranspNFe-&gt;addChild('tpUnidCarga', $dInfUnidCargaTranspNFe['cuc_tipo_unidade_carga']);
            									$infUnidCargaTranspNFe-&gt;addChild('idUnidCarga', $dInfUnidCargaTranspNFe['cuc_identificacao_unidade_carga']);

            									$lacresUnidCargaTranspNFe = $dInfUnidCargaTranspNFe['CteInfNotaFiscalUnidadeCargaLacre'];

            									if(count($lacresUnidCargaTranspNFe) &gt; 0) {
            										foreach($lacresUnidCargaTranspNFe as $dLacUnidCargaTranspNFe) {
            											$lacUnidCargaTranspNFe = $infUnidCargaTranspNFe-&gt;addChild('lacUnidCarga');

            											$lacUnidCargaTranspNFe-&gt;addChild('nLacre', $dLacUnidCargaTranspNFe['ccl_numero_lacre']);
            										}
            									}

            									$infUnidCargaTranspNFe-&gt;addChild('qtdRat', $dInfUnidCargaTranspNFe['cuc_qtd_rateado']);
            								}
            							}
            						}
            					}
            					break;
            			}
            	
            	foreach($config['seg'] as $dSeg) {
            		$seg = $infCTeNorm-&gt;addChild('seg');
            		 
            		$seg-&gt;addChild('respSeg', $dSeg['cns_responsavel_seguro']/*'4'*/);
            		 
            		if($this-&gt;cleaner($dSeg['cns_nome_seguradora'])!="") {
            			$seg-&gt;addChild('xSeg', $dSeg['cns_nome_seguradora']/*'Zurich'*/);
            		}

            		if($this-&gt;cleaner($dSeg['cns_numero_apolice'])!="") {
            			$seg-&gt;addChild('nApol', $dSeg['cns_numero_apolice']);
            		}
            		 
            		if($this-&gt;cleaner($dSeg['cns_numero_averbacao'])!="") {
            			$seg-&gt;addChild('nAver', str_pad($dSeg['cns_numero_averbacao'], 20, STR_PAD_LEFT));
            		}
            		// Padr\E3o da carga \E9 13,2
            		if($this-&gt;cleaner($dSeg['cns_valor_carga_efeito_averbacao'])!="" &amp;&amp; $dSeg['cns_valor_carga_efeito_averbacao'] != '000') {
            			$seg-&gt;addChild('vCarga', $dSeg['cns_valor_carga_efeito_averbacao']/*'47000.00'*/);
            		}
            	}


            	$infModal = $infCTeNorm-&gt;addChild('xs:infModal', $config['infModal']['xs:any']-&gt;asXML());
            	$infModal-&gt;addAttribute('versaoModal', $config['infModal']['versaoModal']);

            	foreach($config['peri'] as $dPeri) {
            		$peri = $infCTeNorm-&gt;addChild('peri');

            		$peri-&gt;addChild('nONU', $dPeri['cno_numero_onu']);
            		$peri-&gt;addChild('xNomeAE', $dPeri['cno_nome_apropriado_embarque_produto']);
            		$peri-&gt;addChild('xClaRisco', $dPeri['cno_classe_subclasse']);
            		$peri-&gt;addChild('grEmb', $dPeri['cno_grupo_embarque']);
            		$peri-&gt;addChild('qTotProd', $dPeri['cno_quantidade_total_produto']);
            		$peri-&gt;addChild('qVolTipo', $dPeri['cno_quantidade_tipo_volume']);
            		$peri-&gt;addChild('pontoFulgor', $dPeri['cno_ponto_fulgor']);
            	}

            	// Se existir carros novos \E9 obrigat\F3rio informar os campos abaixo
            	if ( count($config['veicNovos']) &gt; 0 ) {
            		foreach($config['veicNovos'] as $dVeicNovos) {
            			$veicNovos = $infCTeNorm-&gt;addChild('veicNovos');

            			$veicNovos-&gt;addChild('chassi', $dVeicNovos['cvt_chassi_veiculo']);
            			$veicNovos-&gt;addChild('cCor', $dVeicNovos['cvt_cor_veiculo']);
            			$veicNovos-&gt;addChild('xCor', $dVeicNovos['cvt_descricao_cor']);
            			$veicNovos-&gt;addChild('cMod', $dVeicNovos['cvt_codigo_marca_modelo']);
            			$veicNovos-&gt;addChild('vUnit', $dVeicNovos['cvt_valor_unitario_veiculo']);
            			$veicNovos-&gt;addChild('vFrete', $dVeicNovos['cvt_frete_unitario']);
            		}
            	}

            	$this-&gt;setXml($xml);
            }

            public function save($cte_id) {
            	try {
            		$xml = $this-&gt;getXml();

            		$cteXml = CteXmlTable::getInstance()-&gt;findOneByCteId($cte_id);
            		 
            		if(!$cteXml instanceof CteXml)
            		$cteXml = new CteXml();
            		 
            		$cteXml-&gt;ctx_xml = $xml;
            		$cteXml-&gt;cte_id = $cte_id;
            		$cteXml-&gt;ctx_qem = 1;
            		 
            		$cteXml-&gt;save();

            		return $this;
            	} catch(Exception $e) {
            		exit($e-&gt;getMessage());
            	}
            }
          

            private function limpaEspacos($arranjo) {
            	foreach($arranjo as $indice =&gt; $valor) {
            		if(!is_array($valor)) {
            			$arranjo[$indice] = trim($valor);
            		}
            	}
            	return $arranjo;
            }

            private function cleaner($str) {
            	$str = trim($str);
            	return $str;
            }

            public function calculaDv($cCT) {
            	// omitted code with 11 lines
            }
             
            public function calculaChaveAcesso($cUF, $AAMM, $CNPJ, $mod, $serie, $nCT, $tpEmis, $cCT) {
		// omitted code with 20 lines
            }
}
```

## Crying, Thinking and Planning

As you could see now we have a class with 1054 lines and our challenge is to transform it in testable code.
As a first step I'd recommend you to cry and as a second step we can start to think in what the code does and how can
we break it down in pieces.

In our code we have a couple of method that can be separated in classes instead of methods because they have its own
behavior for example the method setCompl($config) will add to our XML a bunch of nodes depending on some conditions
in this case is used a switch statement either a couple of if's .

```php
class Compl {
public function setCompl($config) {
            	$xml = $this-&gt;getXml();

            	$compl = $xml-&gt;infCte-&gt;addChild('compl');

            	$confComplPrimeiroBloco = array(
					'xCaracAd', 
					'xCaracSer', 
					'xEmi'
					);

					foreach($confComplPrimeiroBloco as $node) {
						$config[$node] = $this-&gt;cleaner($config[$node]);
						if($config[$node]!="")
						$compl-&gt;addChild($node, $config[$node]);
					}

					if($config['fluxo']['xOrig'] != "" || $config['fluxo']['pass'] != "") {
						$fluxo = $compl-&gt;addChild('fluxo');
					}

					if(!empty($config['fluxo']['xOrig'])) {
						$fluxo-&gt;addChild('xOrig', $config['fluxo']['xOrig']);
					}

					if($config['fluxo']['pass'] != "" &amp;&amp; count($config['fluxo']['pass']) &gt; 0) {
						foreach($config['fluxo']['pass'] as $pass) {
							$xpass = $fluxo-&gt;addChild('pass');

							$nodesPass = array(
						'xPass' =&gt; 'cdp_sigla_aeroporto_passagem',
						'xDest' =&gt; 'cdp_sigla_aeroporto_destino',
						'xRota' =&gt; 'cdp_codigo_rota_entrega'
						);
							
						foreach($nodesPass as $nodePass =&gt; $nodePassIndex) {
							$pass[$nodePassIndex] = $this-&gt;cleaner($pass[$nodePassIndex]);
							if($pass[$nodePassIndex]!="")
							$xpass-&gt;addChild($nodePass, $pass[$nodePassIndex]);
						}
						}
					}

					if(count($config['Entrega'])) {
						$Entrega = $compl-&gt;addChild('Entrega');

						$config['Entrega']['tpPer'] = $this-&gt;cleaner($config['Entrega']['tpPer']);

						if($config['Entrega']['tpPer']=="")

						switch($config['Entrega']['tpPer']) {
							case '0' :
								$semData = $Entrega-&gt;addChild('semData');
								$semData-&gt;addChild('tpPer', $config['Entrega']['tpPer']);
								break;
							case '1' :
							case '2' :
							case '3' :
								$comData = $Entrega-&gt;addChild('comData');
								$comData-&gt;addChild('tpPer', $config['Entrega']['tpPer']);
								$config['Entrega']['dProg'] = $this-&gt;cleaner($config['Entrega']['dProg']);

								if($config['Entrega']['dProg']!="")
								$comData-&gt;addChild('tpPer', $config['Entrega']['dProg']);
								
								break;
							case '4' :
								$noPeriodo = $Entrega-&gt;addChild('noPeriodo');
								$noPeriodo-&gt;addChild('tpPer', $config['Entrega']['tpPer']);
								$config['Entrega']['dtIni'] = $this-&gt;cleaner($config['Entrega']['dtIni']);
								if($config['Entrega']['dtIni']!="")
								$noPeriodo-&gt;addChild('dtIni', $config['Entrega']['dtIni']);
								
								$config['Entrega']['dtFim'] = $this-&gt;cleaner($config['Entrega']['dtFim']);
								if($config['Entrega']['dtFim']!="")
								$noPeriodo-&gt;addChild('dtFim', $config['Entrega']['dtFim']);
								
								break;
						}

						$config['Entrega']['tpHor'] = $this-&gt;cleaner($config['Entrega']['tpHor']);

						switch($config['Entrega']['tpHor']) {
							case '0' :
								$semHora = $Entrega-&gt;addChild('semHora');
								$semHora-&gt;addChild('tpHor', $config['Entrega']['tpHor']);
								break;
							case '1' :
							case '2' :
							case '3' :
								$comHora = $Entrega-&gt;addChild('comHora');
								$comHora-&gt;addChild('tpHor', $config['Entrega']['tpHor']);
								$config['Entrega']['hProg'] = $this-&gt;cleaner($config['Entrega']['hProg']);
								if($config['Entrega']['hProg']!="")
								$comHora-&gt;addChild('tpHor', $config['Entrega']['hProg']);
								break;
							case '4' :
								$noInter = $Entrega-&gt;addChild('noInter');
								$noInter-&gt;addChild('tpHor', $config['Entrega']['tpHor']);
								$config['Entrega']['hIni'] = $this-&gt;cleaner($config['Entrega']['hIni']);
								if($config['Entrega']['hIni']!="")
								$noInter-&gt;addChild('hIni', $config['Entrega']['hIni']);
								$config['Entrega']['hFim'] = $this-&gt;cleaner($config['Entrega']['hFim']);
								if($config['Entrega']['hFim']!="")
								$noInter-&gt;addChild('hFim', $config['Entrega']['hFim']);
								break;
						}
					}

					if ( !empty($config['xObs']) ){
						$compl-&gt;addChild('xObs', $config['xObs']);
					}

					if(isset($config['ObsCont']) &amp;&amp; count($config['ObsCont']) &gt; 0) {
						foreach($config['ObsCont'] as $ObsCont) {
							$xObsCont = $compl-&gt;addChild('ObsCont');

							$ObsCont['coic_identificador'] = $this-&gt;cleaner($ObsCont['coic_identificador']);
							if($ObsCont['coic_identificador']!="")
							$xObsCont-&gt;addChild('xCampo', $ObsCont['coic_identificador']);
							$ObsCont['coic_obs'] = $this-&gt;cleaner($ObsCont['coic_obs']);
							if($ObsCont['coic_obs']!="")
							$xObsCont-&gt;addChild('xTexto', $ObsCont['coic_obs']);
						}
					}

					if(isset($config['ObsFisco']) &amp;&amp; count($config['ObsFisco']) &gt; 0) {
						foreach($config['ObsFisco'] as $ObsFisco) {
							$xObsFisco = $compl-&gt;addChild('ObsFisco');

							$ObsFisco['coif_identificador'] = $this-&gt;cleaner($ObsFisco['coif_identificador']);
							if($ObsFisco['coif_identificador']!="")
							$xObsFisco-&gt;addChild('xCampo', $ObsFisco['coif_identificador']);
							$ObsFisco['coif_obs'] = $this-&gt;cleaner($ObsFisco['coif_obs']);
							if($ObsFisco['coif_obs']!="")
							$xObsFisco-&gt;addChild('xTexto', $ObsFisco['coif_obs']);
						}
					}

					$this-&gt;setXml($xml);
					//		}
            }

}
```

It is much better now, isn't it? We've got one class with a one responsibility and very tiny just 146 lines although
this class we have just created could be improved as well but it is a subject to other post.
So we got what we need to do and we have a plan already:

1. Analyze the methods
2. Extract and create new classes

Our last step before go to unit test is substitute the source code to use or new class instead of the currently code
let's find where is the **setComple($config)** and switch our code.

```php
public function setIdeValues($config, $Cte) {
//omitted code
}

public function setCompl($config) {
            	$newCode = new Compl();
		$newCode-&gt;setCompl($config);
}

public function setEmitValues($config) {
//omitted code
}
```

3. Substitute legacy code to your new code

**Now we should test our code in order to see if its still working**, once we
have this confirmation we can now create our unit tests and repeat the cycle
again. As you start refactoring all the legacy using those steps you'll start
to have your source code with better quality, maintainability  and of course
testable.

## It is hard

It is not easy to restructure all the legacy with new code but the result
is amazing. You can fallow the ideology of baby steps in this refactor as well
so as you change the legacy to the new code you test to see if everything is
the same. The trick here is to be patient and change piece by piece.

Basically I tried to show one single way of do it but in the internet you can
find thousands.

1. Analyze the methods
2. Extract and create new classes
3. Substitute legacy code to your new code
4. Make sure everything is working
5. Create your unit tests