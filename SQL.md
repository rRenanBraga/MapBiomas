1

SELECT de.*
FROM queimadas_censipam.tb_deteccao AS de
 
WHERE ST_Within(
    de.geom,
    (SELECT geom 
     FROM bases_auxiliares.ibge_bc250_lim_pais_a 
     WHERE nome = 'Brasil')
)



2


SELECT distinct * FROM "queimadas_censipam"."tb_deteccao" as de
join bases_auxiliares.ibge_bc250_lim_pais_a as ps
on st_intersects(de.geom, ps.geom)
where ps.nome = 'Brasil'



3


select distinct uc.nome_uc as nome, uc.grupo, uc.categoria, uc.esfera,
uc.ano_cria, uc.ato_legal, uc.pl_manejo, uc.ha_total,
st_area(st_transform(uc.geom, 5880)) area_ro, st_intersection(uc.geom, uf.geom) as geom
from bases_auxiliares.mma_cnuc_unidade_conservacao uc
join bases_auxiliares.ibge_bc250_lim_unidade_federacao_a uf
on st_intersects(uc.geom, uf.geom)
where uf.sigla = 'RO'


4 



select distinct uc.terrai_nom as nome_ti, uc.etnia_nome as etnia,
uc.fase_ti, uc.modalidade, uc.superficie as area_total,
st_area(st_transform(uc.geom, 5880)) area_ro, st_intersection(uc.geom, uf.geom) as geom
from bases_auxiliares.funai_terra_indigena uc
join bases_auxiliares.ibge_bc250_lim_unidade_federacao_a uf
on st_intersects(uc.geom, uf.geom)
where uf.sigla = 'RO'



5


select mma.nome_uc, mma.geom, mma.ha_total, mma.codigo_uf
from bases_auxiliares.mma_cnuc_unidade_conservacao as mma
WHERE mma.codigo_uf = 'RONDÔNIA'
 
SELECT ti.geom, ti.terrai_nom, ti.uf_sigla, ti.superficie
FROM bases_auxiliares.funai_terra_indigena as ti
WHERE ti.uf_sigla = 'RO'



6


select SUM(mma.ha_total)
from bases_auxiliares.mma_cnuc_unidade_conservacao as mma
WHERE mma.codigo_uf = 'RONDÔNIA'
 
select SUM(ti.superficie)
from bases_auxiliares.funai_terra_indigena as ti
WHERE ti.uf_sigla = 'RO'




7 - Buffer



with uc as (
select st_buffer(geom, 0.009) as geom_buffer
from bases_auxiliares.mma_cnuc_unidade_conservacao
where id_uc = '233'
)
 
select fc.geom
from queimadas.tb_firms as fc 
join uc 
on st_within(fc.geom, uc.geom_buffer)
where acq_date between '2020-09-20' and '2025-09-20'



8



select
	distinct ev.id_evento as ID,
	max(ev.area_acumulada_ha) as "Área de Inflência (ha)",
	string_agg(DISTINCT uf.sg_uf, ', ') as "UF",
	max(ev.tempo_acumulado_horas)/24 as "Duração (Dias)",
	string_agg(DISTINCT uf.nome, ', ') as "Município",
	max(ev.peso_global_passagem) as "Índice de Severidade",
	max(dt_passagem) as "Data da passagem",
	max(ev.geom_acumulada) as "geom"
from
	queimadas.mv_indicadores_queimadas as ev
join
	bases_auxiliares.ibge_bc250_lim_municipio_a as uf
on
	st_intersects(ev.geom_acumulada, uf.geom)
where
	area_acumulada_ha >= 100
	and dt_passagem >= now() - interval '24 hour'
group by
	ev.id_evento
order by
	"Índice de Severidade" desc
limit 30;



9


select date_trunc('month', ev.dt_minima):: date as mes, count(ev.id), st_collect(ev.geom) 
from queimadas.tb_evento ev
 
join queimadas.tb_escopo_queimadas es
on st_intersects(ev.geom, es.geom)
 
join bases_auxiliares.ibge_bc250_lim_municipio_a mun
on st_intersects(ev.geom,mun.geom)
 
where ev.dt_minima <= '2025-01-01'
and ev.dt_maxima >= '2024-01-01'
and ev.area_km2>1
and ev.id_status_evento != 4
and mun.nome = 'Vilhena'
 
group by date_trunc('month', ev.dt_minima)
order by mes;



10



select ev.*
 
from queimadas.tb_evento ev
 
join bases_auxiliares.funai_terra_indigena ti 
on st_intersects(ev.geom, ti.geom)
 
where ti.terrai_nom = 'Karipuna'
and ev.area_km2 > 1 
and ev.id_status_evento != 4
and ev.dt_minima <= '2025-08-18'
and ev.dt_maxima >= '2025-08-01'



11



select id.id_evento, id.id, id.dt_min_evento, id.dt_max_evento, id.area_acumulada, uf.nome, id.dt_passagem
from queimadas.tb_indicadores_eventos as id
join bases_auxiliares.ibge_bc250_lim_unidade_federacao_a as uf 
on st_intersects(id.geom_acumulada, uf.geom)
join queimadas.tb_escopo_queimadas as es 
on st_intersects (id.geom_acumulada, es.geom)
where id.dt_passagem between '2025-03-01' and '2025-04-01'
and id.area_acumulada>1
and id.id_status_evento != 4
and uf.sigla = 'RO'
