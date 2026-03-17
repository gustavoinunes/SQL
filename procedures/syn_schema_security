DO $$ 
DECLARE 
    -- ==========================================
    -- CONFIGURAÇÃO DE VARIÁVEIS
    v_schema TEXT := 'powerbi';  -- Nome do esquema
    v_owner  TEXT := 'postgres';     -- Novo dono das tabelas
    -- ==========================================
    r RECORD;
BEGIN
    RAISE NOTICE 'Iniciando limpeza e padronização do esquema: %', v_schema;

    -- 1. REVOGAR PRIVILÉGIOS DE TODAS AS ROLES QUE POSSUEM ACESSO (DINÂMICO)
    -- Busca quem tem permissão em tabelas
    FOR r IN (SELECT DISTINCT grantee FROM information_schema.role_table_grants 
              WHERE table_schema = v_schema AND grantee != 'postgres' AND grantee != v_owner) 
    LOOP
        EXECUTE format('REVOKE ALL PRIVILEGES ON ALL TABLES IN SCHEMA %I FROM %I', v_schema, r.grantee);
    END LOOP;

    -- Busca quem tem permissão em sequências
    FOR r IN (SELECT DISTINCT grantee FROM information_schema.role_usage_grants 
              WHERE object_schema = v_schema AND grantee != 'postgres' AND grantee != v_owner) 
    LOOP
        EXECUTE format('REVOKE ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA %I FROM %I', v_schema, r.grantee);
    END LOOP;

    -- 2. LIMPEZA DO PUBLIC E DO PRÓPRIO ESQUEMA
    EXECUTE format('REVOKE ALL ON ALL TABLES IN SCHEMA %I FROM PUBLIC', v_schema);
    EXECUTE format('REVOKE ALL ON ALL SEQUENCES IN SCHEMA %I FROM PUBLIC', v_schema);
    EXECUTE format('REVOKE ALL ON SCHEMA %I FROM PUBLIC', v_schema);

    -- 3. ALTERAR OWNER DE TODAS AS TABELAS
    FOR r IN (SELECT tablename FROM pg_tables WHERE schemaname = v_schema) 
    LOOP
        EXECUTE format('ALTER TABLE %I.%I OWNER TO %I', v_schema, r.tablename, v_owner);
    END LOOP;

    -- 4. ALTERAR OWNER DE TODAS AS SEQUÊNCIAS (Importante para IDs)
    FOR r IN (SELECT relname FROM pg_class c JOIN pg_namespace n ON n.oid = c.relnamespace 
              WHERE n.nspname = v_schema AND c.relkind = 'S') 
    LOOP
        EXECUTE format('ALTER SEQUENCE %I.%I OWNER TO %I', v_schema, r.relname, v_owner);
    END LOOP;

    -- 5. APLICAR AS NOVAS PERMISSÕES PADRÃO (CONFORME SUA REGRA)
    EXECUTE format('GRANT USAGE ON SCHEMA %I TO grupo_bi, grupo_ti', v_schema);
    EXECUTE format('GRANT SELECT ON ALL TABLES IN SCHEMA %I TO grupo_bi', v_schema);
    EXECUTE format('GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA %I TO grupo_ti WITH GRANT OPTION', v_schema);
    
    -- 6. SINCRONIZAR DEFAULT PRIVILEGES (Para tabelas futuras ficarem iguais)
    EXECUTE format('ALTER DEFAULT PRIVILEGES IN SCHEMA %I REVOKE ALL ON TABLES FROM PUBLIC', v_schema);
    EXECUTE format('ALTER DEFAULT PRIVILEGES IN SCHEMA %I GRANT SELECT ON TABLES TO grupo_bi', v_schema);
    EXECUTE format('ALTER DEFAULT PRIVILEGES IN SCHEMA %I GRANT ALL ON TABLES TO grupo_ti WITH GRANT OPTION', v_schema);

    RAISE NOTICE 'Sincronização concluída com sucesso para o esquema %.', v_schema;
END $$;
