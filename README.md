const puppeteer = require('puppeteer');
const ExcelJS = require('exceljs');
const fs = require('fs');

const LOGIN_URL = 'https://pje1g.trf5.jus.br/pje/login.seam?loginComCertificado=false';
const USERNAME = '01614686337';
const PASSWORD = 'senhalladv123';

async function lerCPFsDoExcel(arquivo) {
    const workbook = new ExcelJS.Workbook();
    await workbook.xlsx.readFile(arquivo);
    const worksheet = workbook.getWorksheet('Plan1');
    const cpfs = [];

    worksheet.eachRow((row, rowNumber) => {
        if (rowNumber > 1) {
            const cpf = row.getCell(1).text.trim();
            if (cpf) cpfs.push(cpf);
        }
    });

    return cpfs;
}

async function escreverResultados(resultados, nomeArquivo) {
    const workbook = new ExcelJS.Workbook();
    const worksheet = workbook.addWorksheet('Resultados');

    worksheet.columns = [
        { header: 'CPF', key: 'cpf', width: 20 },
        { header: 'Processo Encontrado', key: 'status', width: 30 }
    ];

    resultados.forEach(item => {
        worksheet.addRow(item);
    });

    await workbook.xlsx.writeFile(nomeArquivo);
    console.log(Resultados salvos em: ${nomeArquivo});
}

async function consultaCpf(browser, cpf) {
    const page = await browser.newPage();
    try {
        await page.goto(LOGIN_URL, { waitUntil: 'networkidle2' });

        await page.type('#loginApplication\\:username', USERNAME);
        await page.type('#loginApplication\\:password', PASSWORD);
        await page.click('#loginApplication\\:loginButton');

        await page.waitForNavigation({ waitUntil: 'networkidle2' });

        // Aqui é necessário adaptar para o local exato de busca do CPF dentro do sistema do TRF5
        // Exemplo genérico:
        // await page.goto('https://pje1g.trf5.jus.br/pje/consulta', { waitUntil: 'networkidle2' });
        // await page.type('#cpfSearchInput', cpf);
        // await page.click('#searchButton');
        // await page.waitForSelector('.resultado');

        const temProcesso = Math.random() > 0.5; // Simulação
        return { cpf, status: temProcesso ? 'Sim' : 'Não' };

    } catch (err) {
        console.error(Erro ao consultar CPF ${cpf}: ${err.message});
        return { cpf, status: 'Erro' };
    } finally {
        await page.close();
    }
}

(async () => {
    const browser = await puppeteer.launch({ headless: true });
    const cpfs = await lerCPFsDoExcel('cpfs.xlsx');
    const resultados = [];

    for (const cpf of cpfs) {
        const resultado = await consultaCpf(browser, cpf);
        resultados.push(resultado);
        console.log(Consultado: ${cpf} - ${resultado.status});
    }

    await browser.close();
    await escreverResultados(resultados, 'resultado_cpfs.xlsx');
})();
