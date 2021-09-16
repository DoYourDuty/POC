# POC


```csharp
public static void ConvertPDFtoTiff(string orgFileName)
        {
            FileInfo fi = new FileInfo(orgFileName);
            fi.CopyTo(fi.FullName + "temp");
            string filename = fi.FullName + "temp";
            

            PDFLibNet32.PDFWrapper doc = new PDFLibNet32.PDFWrapper();
            doc.LoadPDF(filename);

            System.Drawing.Imaging.Encoder enc = System.Drawing.Imaging.Encoder.SaveFlag;
            ImageCodecInfo info = null;
            info = (from ie in ImageCodecInfo.GetImageEncoders()
                    where ie.MimeType == "image/tiff"
                    select ie).FirstOrDefault();
            EncoderParameters encoderparams = new EncoderParameters(1);
            encoderparams.Param[0] = new EncoderParameter(enc, (long)EncoderValue.MultiFrame);

            Bitmap buffer = null;
            Bitmap buffer1 = null;
            for (int i = 0; i < doc.PageCount; i++)
            {

                doc.CurrentPage = i + 1;
                doc.CurrentX = 0;
                doc.CurrentY = 0;

                doc.RenderPage(IntPtr.Zero);

                buffer = new Bitmap(doc.PageWidth, doc.PageHeight);

                doc.ClientBounds = new Rectangle(0, 0, doc.PageWidth, doc.PageHeight);
                using (var g = Graphics.FromImage(buffer))
                {
                    var hdc = g.GetHdc();
                    try
                    {
                        doc.DrawPageHDC(hdc);
                    }
                    finally
                    {
                        g.ReleaseHdc();
                    }
                }

                if (i == 0)
                {
                    buffer1 = (Bitmap)buffer;
                    // create an image to draw the page into

                    //Save the bitmap
                    buffer1.Save(orgFileName, info, encoderparams);
                    encoderparams.Param[0] = new EncoderParameter(enc, (long)EncoderValue.FrameDimensionPage);
                }
                else
                {
                    //add another image
                    //Repeat this to add multiple images
                    buffer1.SaveAdd(buffer, encoderparams);
                }
            }
            //close file
            encoderparams.Param[0] = new EncoderParameter(enc, (long)EncoderValue.Flush);
            buffer1.SaveAdd(encoderparams);

            doc.Dispose();
            return;
        }
```
