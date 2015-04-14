OBO = http://purl.obolibrary.org/obo
CATALOG = catalog-v001.xml

omia.owl: omia-edit.obo
	owltools --catalog-xml $(CATALOG) $< $(OBO)/hp.owl $(OBO)/mp.owl $(OBO)/upheno/hp-mp/mp_hp-align-equiv.owl \
	 --add-imports-from-supports --assert-inferred-subclass-axioms --useIsInferred --allowEquivalencies \
	  -o $@ \
	  -o -f obo omia.obo \
	 --extract-bridge-ontologies -d . -s omia -x \
	  -o core.owl \
	  -o -f obo core.obo
.PRECIOUS: omia.owl

omia.obo: omia.owl
	owltools  --catalog-xml $(CATALOG) $< -o -f obo $@

importer.owl: omia.owl
	owltools $< --extract-bridge-ontologies -d . -s omia -x -o file://`pwd`/$@

oort: omia-edit.obo
	ontology-release-runner --prefix $(OBO)/OMIA_ --catalog-xml $(CATALOG) --allow-overwrite --force --outdir oort --reasoner elk $< $(OBO)/hp.owl $(OBO)/mp.owl $(OBO)/upheno/hp-mp/mp_hp-align-equiv.owl $(OBO)/doid.owl
# owltools  oort/omia.owl --merge-equivalent-classes -f $OBO/HP -f $OBO/MP -f $OBO/DOID -f $OBO/SCTID -t $OBO/OMIA -o file://`pwd`/z.owl -o -f obo z.obo

bridges: omia.obo

omia-genes.txt:
	wget 'http://omia.angis.org.au/curate/causal_mutations/?format=gene_table' -O $@
.PRECIOUS: omia-genes.txt

omia-phene-id-name.txt: omia-genes.txt
	cut -f3,6 $< | perl -npe 's/^/OMIA:/' > $@

omia-s1.obo: omia-phene-id-name.txt
	tbl2obo.pl $< | perl -npe 's/name: (\w)/"name: ".lc($$1)/e' > $@

omim-rels.obo: omia.xml
	./parse-omim-xrefs.pl $< | tbl2obolinks.pl --rel similar_to > $@

align-%.txt: omia.obo
	blip-findall -consult nlp_filter.pro -u metadata_nlp -u metadata_nlp_disorder_hook -goal index_entity_pair_label_match -i $< -r $* "entity_pair_label_reciprocal_best_intermatch(A,B,S)" -use_tabs -label -no_pred > $@.tmp && grep -v UMLS: $@.tmp |  sort -u > $@
.PRECIOUS: align-%.txt

