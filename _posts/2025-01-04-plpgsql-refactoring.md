---
layout: post
title: Refactoring in PL/pgSQL
tags: Evergreen Postgres refactoring testing
categories: Systems
---

While working with some older PL/pgSQL routines in Evergreen
ILS, I wanted to refactor before making changes in the interest
of Kent Beck's "make the change easy (warning: this may be hard),
then make the easy change".  But... my brain is used
to refactoring in object oriented paradigms, and it wasn't
immediately obvious to me how to transfer that knowledge to
PL/pgSQL.

Fortunately, the 2019 edition of Martin Fowler's Refactoring book
has a lot to offer in non-OO paradigms!  I've started working
through the different refactorings in that book,
using code examples from a real world application (Evergreen ILS),
with some [pgTAP tests](https://pgtap.org/)
at my back.  Here is the first batch,
some refactorings from Chapter 6: A First Set of Refactorings.

Note: these are for practice and demonstration only, I don't
think all of these examples warrant refactoring, or that the
refactoring presented is the appropriate one.

Also, I'm presenting all of the code examples verbatim as they
were in the source code at the time I looked at them.  Apologies
in advance for the no doubt huge amount of domain- and application-
specific assumptions, I hope these are useful regardless.

## Extract function

PgTAP assertion:
```
 SELECT is(
    (SELECT authority.extract_thesaurus('<record><leader>      z</leader><controlfield tag="008">           z</controlfield><datafield tag="040"><subfield code="f">Hello</subfield></datafield></record>')),
    'Hello',
    'It can extract 040f from an authority record when 008/11 is z');
```

Original:
```
CREATE OR REPLACE FUNCTION authority.extract_thesaurus( marcxml TEXT ) RETURNS TEXT AS $func$
DECLARE
    thes_code TEXT;
BEGIN
    thes_code := vandelay.marc21_extract_fixed_field(marcxml,'Subj');
    IF thes_code IS NULL THEN
        thes_code := '|';
    ELSIF thes_code = 'z' THEN
        thes_code := COALESCE( oils_xpath_string('//*[@tag="040"]/*[@code="f"][1]', marcxml), 'z' );
    ELSE
        SELECT code INTO thes_code FROM authority.thesaurus WHERE short_code = thes_code;
        IF NOT FOUND THEN
            thes_code := '|'; -- default
        END IF;
    END IF;
    RETURN thes_code;
END;
$func$ LANGUAGE PLPGSQL STABLE STRICT;
```

Let's extract the contents of the `ELSIF thes_code = 'z' THEN` branch into its own
function:

```
CREATE OR REPLACE FUNCTION authority.extract_thesaurus_from_040( marcxml TEXT ) RETURNS TEXT as $func$
BEGIN
RETURN COALESCE( oils_xpath_string('//*[@tag="040"]/*[@code="f"][1]', marcxml), 'z' );
END;
$func$ LANGUAGE PLPGSQL IMMUTABLE;

CREATE OR REPLACE FUNCTION authority.extract_thesaurus( marcxml TEXT ) RETURNS TEXT AS $func$
DECLARE
    thes_code TEXT;
BEGIN
    thes_code := vandelay.marc21_extract_fixed_field(marcxml,'Subj');
    IF thes_code IS NULL THEN
        thes_code := '|';
    ELSIF thes_code = 'z' THEN
        thes_code := authority.extract_thesaurus_from_040(marcxml);
    ELSE
        SELECT code INTO thes_code FROM authority.thesaurus WHERE short_code = thes_code;
        IF NOT FOUND THEN
            thes_code := '|'; -- default
        END IF;
    END IF;
    RETURN thes_code;
END;
$func$ LANGUAGE PLPGSQL STABLE STRICT;
```

## Extract variable

The following example is a function that is used to generate test data.  The PgTAP assertion:

```
SELECT evergreen.create_aou_address(1, '123 great street'::text, NULL, 'Detroit'::text, 'Oregon'::text, ''::text, '12345'::text, NULL);
SELECT is(
    (SELECT street1::text FROM actor.org_unit aou INNER JOIN actor.org_address oa ON oa.id=aou.ill_address WHERE aou.id =1),
    '123 great street',
    'It sets the org unit address to the provided values'
);
```
The original:

```
CREATE FUNCTION evergreen.create_aou_address
    (owning_lib INTEGER, street1 TEXT, street2 TEXT, city TEXT, state TEXT, country TEXT,
     post_code TEXT, address_type TEXT)
RETURNS void AS $$
BEGIN
    INSERT INTO actor.org_address (org_unit, street1, street2, city, state, country, post_code)
        VALUES ($1, $2, $3, $4, $5, $6, $7);
    
    IF $8 IS NULL THEN
	UPDATE actor.org_unit SET holds_address = currval('actor.org_address_id_seq'), ill_address = currval('actor.org_address_id_seq'), billing_address = currval('actor.org_address_id_seq'), mailing_address = currval('actor.org_address_id_seq') WHERE id = $1;
    END IF;
    IF $8 ~ 'holds' THEN
	UPDATE actor.org_unit SET holds_address = currval('actor.org_address_id_seq') WHERE id = $1;
    END IF;
    IF $8 ~ 'interlibrary' THEN
	UPDATE actor.org_unit SET ill_address = currval('actor.org_address_id_seq') WHERE id = $1;
    END IF;
    IF $8 ~ 'billing' THEN
	UPDATE actor.org_unit SET billing_address = currval('actor.org_address_id_seq') WHERE id = $1;
    END IF;
    IF $8 ~ 'mailing' THEN
	UPDATE actor.org_unit SET mailing_address = currval('actor.org_address_id_seq') WHERE id = $1;
    END IF;
END
$$ LANGUAGE PLPGSQL;
```

`currval('actor.org_address_id_seq')` is not that mysterious, but we can do better!  We
extract this into an explanatory variable. Note that we need to
declare the variable in the `DECLARE` block.

```
CREATE OR REPLACE FUNCTION evergreen.create_aou_address
    (owning_lib INTEGER, street1 TEXT, street2 TEXT, city TEXT, state TEXT, country TEXT,
     post_code TEXT, address_type TEXT)
RETURNS void AS $$
DECLARE
  new_address_id integer;
BEGIN
    INSERT INTO actor.org_address (org_unit, street1, street2, city, state, country, post_code)
        VALUES ($1, $2, $3, $4, $5, $6, $7);
    new_address_id = currval('actor.org_address_id_seq');
    
    IF $8 IS NULL THEN
	UPDATE actor.org_unit SET holds_address = new_address_id, ill_address = new_address_id, billing_address = new_address_id, mailing_address = new_address_id WHERE id = $1;
    END IF;
    IF $8 ~ 'holds' THEN
	UPDATE actor.org_unit SET holds_address = new_address_id WHERE id = $1;
    END IF;
    IF $8 ~ 'interlibrary' THEN
	UPDATE actor.org_unit SET ill_address = new_address_id WHERE id = $1;
    END IF;
    IF $8 ~ 'billing' THEN
	UPDATE actor.org_unit SET billing_address = new_address_id WHERE id = $1;
    END IF;
    IF $8 ~ 'mailing' THEN
	UPDATE actor.org_unit SET mailing_address = new_address_id WHERE id = $1;
    END IF;
END
$$ LANGUAGE PLPGSQL;
```

## Inline variable

The PgTAP assertion (note that this relies on Evergreen's concerto test data set):

```
SELECT results_eq('SELECT tag::text FROM authority.flatten_marc(1)',
    $$VALUES ('LDR'), ('001'), ('003'), ('005'), ('008'), ('035'), ('100'), ('100'), ('901'), ('901')$$,
    'It can extract the fields from an authority marc record');
```

The original:
```
CREATE OR REPLACE FUNCTION authority.flatten_marc ( rid BIGINT ) RETURNS SETOF authority.full_rec AS $func$
DECLARE
	auth	authority.record_entry%ROWTYPE;
	output	authority.full_rec%ROWTYPE;
	field	RECORD;
BEGIN
	SELECT INTO auth * FROM authority.record_entry WHERE id = rid;

	FOR field IN SELECT * FROM vandelay.flatten_marc( auth.marc ) LOOP
		output.record := rid;
		output.ind1 := field.ind1;
		output.ind2 := field.ind2;
		output.tag := field.tag;
		output.subfield := field.subfield;
		output.value := field.value;

		RETURN NEXT output;
	END LOOP;
END;
$func$ LANGUAGE PLPGSQL;
```

The best way I thought of to inline that `auth` variable is with a `LATERAL`:

```
CREATE OR REPLACE FUNCTION authority.flatten_marc ( rid BIGINT ) RETURNS SETOF authority.full_rec AS $func$
DECLARE
	output	authority.full_rec%ROWTYPE;
	field	RECORD;
BEGIN
	FOR field IN SELECT flattened.* FROM authority.record_entry are, LATERAL vandelay.flatten_marc(are.marc) flattened WHERE id=rid LOOP
		output.record := rid;
		output.ind1 := field.ind1;
		output.ind2 := field.ind2;
		output.tag := field.tag;
		output.subfield := field.subfield;
		output.value := field.value;

		RETURN NEXT output;
	END LOOP;
END;
$func$ LANGUAGE PLPGSQL;
```

## Change function declaration

In Martin Fowler's 2019 edition, this refactoring includes both
renaming a function and changing its parameters, so I decided to
do one of each.

### Changing function name

pgTAP assertion (relies on the concerto test data set):

```
PREPARE allocate_some AS INSERT INTO acq.fund_allocation_percent (funding_source, org, fund_code, percent, allocator) VALUES (1, 1, 'AD', 70, 1);
PREPARE too_much AS INSERT INTO acq.fund_allocation_percent (funding_source, org, fund_code, percent, allocator) VALUES (1, 2, 'AV', 40, 1);
SELECT lives_ok('allocate_some');
SELECT throws_like('too_much', 'Total percentages exceed 100 for funding_source 1');
```

Original function and trigger, which have a name that is not super
clear and unfortunately has a vulgar meaning:

```
CREATE OR REPLACE FUNCTION acq.fap_limit_100()
RETURNS TRIGGER AS $$
DECLARE
--
total_percent numeric;
--
BEGIN
    SELECT
        sum( percent )
    INTO
        total_percent
    FROM
        acq.fund_allocation_percent AS fap
    WHERE
        fap.funding_source = NEW.funding_source;
    --
    IF total_percent > 100 THEN
        RAISE EXCEPTION 'Total percentages exceed 100 for funding_source %',
            NEW.funding_source;
    ELSE
        RETURN NEW;
    END IF;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER acqfap_limit_100_trig
    AFTER INSERT OR UPDATE ON acq.fund_allocation_percent
    FOR EACH ROW EXECUTE PROCEDURE acq.fap_limit_100();
```

Refactored!

```
CREATE OR REPLACE FUNCTION acq.funding_allocation_percent_does_not_exceed_100()
RETURNS TRIGGER AS $$
DECLARE
total_percent numeric;
BEGIN
    SELECT
        sum( percent )
    INTO
        total_percent
    FROM
        acq.fund_allocation_percent AS fap
    WHERE
        fap.funding_source = NEW.funding_source;
    IF total_percent > 100 THEN
        RAISE EXCEPTION 'Total percentages exceed 100 for funding_source %',
            NEW.funding_source;
    ELSE
        RETURN NEW;
    END IF;
END;
$$ LANGUAGE plpgsql;


CREATE OR REPLACE TRIGGER acqfap_limit_100_trig
    AFTER INSERT OR UPDATE ON acq.fund_allocation_percent
    FOR EACH ROW EXECUTE PROCEDURE acq.funding_allocation_percent_does_not_exceed_100();
```

### Adding a parameter

pgTAP assertion:

```
SELECT is(
    (SELECT search.symspell_transfer_casing('DrAGoN', 'beasts')),
    'BeAStS',
    'It applies the case pattern from the first string to the second');
```

The original:

```
CREATE OR REPLACE FUNCTION search.symspell_transfer_casing ( withCase TEXT, withoutCase TEXT )
RETURNS TEXT AS $F$
DECLARE
    woChars TEXT[];
    curr    TEXT;
    ind     INT := 1;
BEGIN
    woChars := regexp_split_to_array(withoutCase,'');
    FOR curr IN SELECT x FROM regexp_split_to_table(withCase, '') x LOOP
        IF curr = evergreen.uppercase(curr) THEN
            woChars[ind] := evergreen.uppercase(woChars[ind]);
        END IF;
        ind := ind + 1;
    END LOOP;
    RETURN ARRAY_TO_STRING(woChars,'');
END;
$F$ LANGUAGE PLPGSQL STRICT IMMUTABLE;
```

Refactored, along with a deprecation warning when users call the old
API.

```
CREATE OR REPLACE FUNCTION search.symspell_transfer_casing ( withCase TEXT, withoutCase TEXT, uselessNewParam TEXT )
RETURNS TEXT AS $F$
DECLARE
    woChars TEXT[];
    curr    TEXT;
    ind     INT := 1;
BEGIN
    woChars := regexp_split_to_array(withoutCase,'');
    FOR curr IN SELECT x FROM regexp_split_to_table(withCase, '') x LOOP
        IF curr = evergreen.uppercase(curr) THEN
            woChars[ind] := evergreen.uppercase(woChars[ind]);
        END IF;
        ind := ind + 1;
    END LOOP;
    RETURN ARRAY_TO_STRING(woChars,'');
END;
$F$ LANGUAGE PLPGSQL STRICT IMMUTABLE;


CREATE OR REPLACE FUNCTION search.symspell_transfer_casing ( withCase TEXT, withoutCase TEXT )
RETURNS TEXT AS $F$
BEGIN
    RAISE WARNING 'Calling search.symspell_transfer_casing with two params is deprecated, please add a third param';
    RETURN search.symspell_transfer_casing(withCase, withoutCase, 'ignoreme');
END;
$F$ LANGUAGE PLPGSQL STRICT IMMUTABLE;
```
