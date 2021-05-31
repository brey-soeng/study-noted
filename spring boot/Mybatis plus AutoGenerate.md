AutoGenerator is the code generator of MyBatis-Plus. Through AutoGenerator, you can quickly generate code for various modules such as Entity, Mapper, Mapper XML, Service, Controller, etc., which greatly improves development efficiency.

# Add dependency

1. Add code generator dependency

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.1</version>
</dependency>
```

2. Add template engine dependency. MyBatis-Plus supports Velocity (default), Freemarker, Beetl. Users can choose the template engine they are familiar with. If it does not meet your requirements, you can use a custom template engine.

   - Velocity (default):

   ```xml
   <dependency>
       <groupId>org.apache.velocity</groupId>
       <artifactId>velocity-engine-core</artifactId>
       <version>2.3</version>
   </dependency>
   ```

   - Freemarker:

   ```xml
   <dependency>
       <groupId>org.freemarker</groupId>
       <artifactId>freemarker</artifactId>
       <version>2.3.31</version>
   </dependency>
   ```

   - Beetl

   ```xml
   <dependency>
       <groupId>com.ibeetl</groupId>
       <artifactId>beetl</artifactId>
       <version>3.3.2.RELEASE</version>
   </dependency>
   ```

note! If you choose a non-default engine, you need to set the template engine in AutoGenerator.

```java
AutoGenerator generator = new AutoGenerator();

// set freemarker engine
generator.setTemplateEngine(new FreemarkerTemplateEngine());

// set beetl engine
generator.setTemplateEngine(new BeetlTemplateEngine());

// set custom engine (reference class is your custom engine class)
generator.setTemplateEngine(new CustomTemplateEngine());
```

### Write configuration

The code generator of MyBatis-Plus provides a large number of custom parameters for users to choose, which can meet the needs of most people.

```
import com.baomidou.mybatisplus.core.exceptions.MybatisPlusException;
import com.baomidou.mybatisplus.core.toolkit.StringPool;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.InjectionConfig;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.builder.ConfigBuilder;
import com.baomidou.mybatisplus.generator.config.po.TableInfo;
import com.baomidou.mybatisplus.generator.config.rules.FileType;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;
import org.junit.platform.commons.util.StringUtils;
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class Generator {
    public static String scanner(String tip) {
        Scanner scanner = new Scanner(System.in);
        StringBuilder help = new StringBuilder();
        help.append(tip + "：");
        System.out.println(help.toString());
        if (scanner.hasNext()) {
            String ipt = scanner.next();
            if (StringUtils.isNotBlank(ipt)) {
                return ipt;
            }
        }
        throw new MybatisPlusException("please right tip" + tip + "！");
    }

    public static void main(String[] args) {
        // codeGenerator
        AutoGenerator mpg = new AutoGenerator();

        // global config
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");
        gc.setAuthor("SOENG BREY");
        gc.setOpen(false);
        // gc.setSwagger2(true); Entity attribute Swagger2 annotation
        mpg.setGlobalConfig(gc);

        // datasourceConfig
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setUrl("jdbc:mysql://localhost:3306/springdemo?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC&useSSL=false");
        // dsc.setSchemaName("public");
        dsc.setDriverName("com.mysql.cj.jdbc.Driver");
        dsc.setUsername("root");
        dsc.setPassword("");
        mpg.setDataSource(dsc);

        // package config
        PackageConfig pc = new PackageConfig();
        pc.setModuleName(scanner("input your packagename"));
        pc.setParent("com.sb.springbootdev");
        mpg.setPackageInfo(pc);

        // yourselftconfig
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };

        // If the template engine is freemarker
        String templatePath = "/templates/mapper.xml.ftl";
       // If the template engine is velocity
        // String templatePath = "/templates/mapper.xml.vm";

        //output config
        List<FileOutConfig> focList = new ArrayList<>();
        // Custom configuration will be output first
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // Customize the output file name. If you set the prefix and suffix in Entity, please note that the name of xml will change accordingly! !
                return projectPath + "/src/main/resources/mapper/" + pc.getModuleName()
                        + "/" + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });
        // override return true
        cfg.setFileCreate(new IFileCreate() {
            @Override
            public boolean isCreate(ConfigBuilder configBuilder, FileType fileType, String filePath) {
//                //Determine whether the custom folder needs to be created
//                checkDir("Call the directory created by the default method, and use the custom directory");
//                if (fileType == FileType.MAPPER) {
//                   // The mapper file has been generated and it is judged to exist, and I do not want to regenerate it and return false
//                    return !new File(filePath).exists();
//                }
                // override old file
                return true;
            }
        });

        cfg.setFileOutConfigList(focList);
        mpg.setCfg(cfg);

        // model config template
        TemplateConfig templateConfig = new TemplateConfig();

        // selft  model config  template
        //Specify the custom template path, be careful not to bring .ftl/.vm, it will be automatically identified according to the template engine used
        // templateConfig.setEntity("templates/entity2.java");
        // templateConfig.setService();
        // templateConfig.setController();

        templateConfig.setXml(null);
        mpg.setTemplate(templateConfig);

        // strategy config
        StrategyConfig strategy = new StrategyConfig();
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
//        strategy.setSuperEntityClass("Your own parent entity, you don't need to set it if you don't have it!");
        strategy.setEntityLombokModel(true);
        strategy.setRestControllerStyle(true);// front-back end project
        // Public parent
//        strategy.setSuperControllerClass("Your own parent controller, you don’t need to set it if you don’t have it!");
        // Public fields written in the parent class
//      strategy.setSuperEntityColumns("id");
        strategy.setInclude(scanner("tablename，use , to split").split(","));
        strategy.setControllerMappingHyphenStyle(true);
        strategy.setTablePrefix(pc.getModuleName() + "_");
        mpg.setStrategy(strategy);
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());
        mpg.execute();
    }
}
```

https://baomidou.com/guide/generator.html#%E6%B7%BB%E5%8A%A0%E4%BE%9D%E8%B5%96

