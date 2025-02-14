# Note
This repository is outdated so use it for reference only.
Use the official and latest repository at https://github.com/tonimartir/reportman

# reportman
.Net Project of https://reportman.sourceforge.io

This is a version compiled in VS 2019 from the "reportmannet_source_3_4_6.zip" download page. Note: Download is not available as of now.

https://sourceforge.net/p/reportman/activity/?page=0&limit=100#615ad5c647012700a256ad7c

Project upgraded to .Net Framework 4.8

## Enhancements
* Added move buttons (left, right, up, down) in designer to precisely move all selected controls (across multiple sections) with snap to grid functionality.
* Preview Form Title will show text "Preview" when preview is called as ModalForm
* Added new properties in Label and Expression control for highlighting: Highlight Condition, Highlight Font, Highlight Font Color, Highlight Back Color etc. If condition is True Highlight Font properties will be used.

## Corrections
* After selecting Font Style from Property, the FontName was getting reset to Default OS font. Now it will preserve the selected font.
* Solved runtime error when multiple Image objects where selected and Image property was null (no image selected in the control)
* Solved runtime error which occurred onPaint of selection rectangle of (mainly Label) control.

Usage from a NET Core 3.1 Web API

    using Microsoft.AspNetCore.Mvc;
    using System.IO;

    using Reportman.Drawing;
    using Reportman.Reporting;
    using System.Text;
    using System.Threading.Tasks;

    [Route("api/[controller]")]
    [ApiController]
    public class PdfController : ControllerBase
    {
        [HttpGet]
        [Route("ot/{idOT}")]
        public async Task<ActionResult> OT(string idOT)
        {
            Report rp = new Report();
            rp.LoadFromFile("C:\\Users\\cesar\\source\\repos\\SCORE\\HemoecoAPI\\ot-net.rep");
            rp.ConvertToDotNet();
            // FixReport
            rp.AsyncExecution = false;
            PrintOutPDF printpdf = new PrintOutPDF();

            Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);

            // Perform the conversion from one encoding to the other.
            if (rp.Params.Count > 0)
            {
                // Unicode doesn't work
                string titulo = $"Orden de trabajo {idOT}"; ; // "你好"; // $"Orden de trabajo {idOT}";
                byte[] unicodeBytes = Encoding.Convert(Encoding.ASCII, Encoding.UTF8, Encoding.ASCII.GetBytes(titulo));
                rp.Params[0].Value = Encoding.UTF8.GetString(unicodeBytes);
            }
            printpdf.FileName = $"ot-{idOT}.pdf";
            printpdf.Compressed = false;
            
            if (printpdf.Print(rp.MetaFile))
            {
                // Download Files using Web API. Changhui Xu. https://codeburst.io/download-files-using-web-api-ae1d1025f0a9
                var bytes = await System.IO.File.ReadAllBytesAsync(printpdf.FileName);
                return File(bytes, "application/pdf", printpdf.FileName);
            }

            // todo: Return file with failure indicated
            return null;
        }
      }
    }
        
